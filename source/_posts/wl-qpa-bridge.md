---
title: treeland 与Qt QPA对接分析
date: 2024/06/19 17:22:25
comments: true
toc: true
tags:
  - linux Desktop Environment
categories:
  - treeland
---

## QPA 插件

|**类名**|**描述**|
|:----|:----|
|QWlrootsOutputWindow|Qt窗口平台抽象实现，实现Qt中QWindow部分功能 |
|QWlrootsIntegration|Qt QPA平台插件实现，对接wlroots输入输出到Qt |
|QWlrootsCursor|Qt QPA光标平台抽象实现，对接wlr_cursor 前端部分功能到Qcursor（代码待完善） |
|QWlrootsScreen|Qt QPA屏幕平台抽象实现，对接 wlr_output 前端部分功能到 Qscreen |

其中如下图1.1所示是将 wlr_renderer 转为 QWlrootsIntegration 的流程：
![1.1 QPA硬件渲染对接流程](/img/treeland/wlrc-qpa-rhi.drawio.svg)

wlroots支持Pixman、OpenGLES、Vulkan渲染，其与QQuickWindow更加详细的渲染绑定如下图：
```c++
QQuickRenderTarget WRenderHelper::acquireRenderTarget(QQuickRenderControl *rc, QWBuffer *buffer)
{
    W_D(WRenderHelper);
    Q_ASSERT(buffer);

    if (d->size.isEmpty())
        return {};

    for (int i = 0; i < d->buffers.count(); ++i) {
        auto data = d->buffers[i];
        if (data->buffer == buffer) {
            d->lastBuffer = data;
            return data->renderTarget;
        }
    }

    std::unique_ptr<BufferData> bufferData(new BufferData);
    bufferData->buffer = buffer;
    auto texture = QWTexture::fromBuffer(d->renderer, buffer);

    QQuickRenderTarget rt;

    if (wlr_renderer_is_pixman(d->renderer->handle())) {
        pixman_image_t *image = wlr_pixman_texture_get_image(texture->handle());
        void *data = pixman_image_get_data(image);
        if (bufferData->paintDevice.constBits() != data)
            bufferData->paintDevice = WTools::fromPixmanImage(image, data);
        Q_ASSERT(!bufferData->paintDevice.isNull());
        rt = QQuickRenderTarget::fromPaintDevice(&bufferData->paintDevice);
    }
#ifdef ENABLE_VULKAN_RENDER
    else if (wlr_renderer_is_vk(d->renderer->handle())) {
        wlr_vk_image_attribs attribs;
        wlr_vk_texture_get_image_attribs(texture->handle(), &attribs);
        rt = QQuickRenderTarget::fromVulkanImage(attribs.image, attribs.layout, attribs.format, d->size);
    }
#endif
    else if (wlr_renderer_is_gles2(d->renderer->handle())) {
        wlr_gles2_texture_attribs attribs;
        wlr_gles2_texture_get_attribs(texture->handle(), &attribs);

        rt = QQuickRenderTarget::fromOpenGLTexture(attribs.tex, d->size);
        rt.setMirrorVertically(true);
    }

    delete texture;
    bufferData->renderTarget = rt;

    if (QSGRendererInterface::isApiRhiBased(getGraphicsApi(rc))) {
        if (!rt.isNull()) {
            // Force convert to Rhi render target
            if (!d->ensureRhiRenderTarget(rc, bufferData.get()))
                bufferData->renderTarget = {};
        }

        if (bufferData->renderTarget.isNull())
            return {};
    }

    connect(buffer, SIGNAL(beforeDestroy()),
            this, SLOT(onBufferDestroy()), Qt::UniqueConnection);

    d->buffers.append(bufferData.release());
    d->lastBuffer = d->buffers.last();

    return d->buffers.last()->renderTarget;
}
```

屏幕热插拔
```c++
// 显示器插入
void WBackendPrivate::on_new_output(QWOutput *output)
{
    auto woutput = new WOutput(output, q_func());

    outputList << woutput;
    QWlrootsIntegration::instance()->addScreen(woutput); // 对接QPA

    woutput->safeConnect(&QWOutput::beforeDestroy, q_func()->server(), [this, output] {
        on_output_destroy(output);
    });

    q_func()->outputAdded(woutput);
}

// 显示器拔出
void WBackendPrivate::on_output_destroy(QWOutput *output)
{
    for (int i = 0; i < outputList.count(); ++i) {
        if (outputList.at(i)->handle() == output) {
            auto device = outputList.takeAt(i);
            q_func()->outputRemoved(device);
            delete device;
            return;
        }
    }
}

void WBackend::outputRemoved(WOutput *output)
{
    QWlrootsIntegration::instance()->removeScreen(output); // 对接QPA
}
```

输入设备热插拔
```c++
// 输入设备插入
void WSeatPrivate::attachInputDevice(WInputDevice *device)
{
    W_Q(WSeat);
    device->setSeat(q);
    auto qtDevice = QWlrootsIntegration::instance()->addInputDevice(device, name);  // 对接QPA
    Q_ASSERT(qtDevice);

    ....
}

// 输入设备拔出
void WSeatPrivate::detachInputDevice(WInputDevice *device)
{
    if (cursor && device->type() == WInputDevice::Type::Pointer)
        cursor->detachInputDevice(device);

    bool ok = QWlrootsIntegration::instance()->removeInputDevice(device); // 对接QPA
    Q_ASSERT(ok);
}
```