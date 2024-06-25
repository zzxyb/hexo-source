---
title: treeland 初始化分析
date: 2024/06/19 17:22:25
comments: true
toc: true
tags:
  - linux Desktop Environment
categories:
  - treeland
---

## 初始化流程
treeland 初始化与 QML 应用代码初始化类似，只是对于 QPA 的初始化 treeland 是显式地，其目的是使其加载 waylib项目中 的 QWlrootsIntergration 插件，将 wlroots 相关功能与 Qt 对接。
![1.1 treeland初始化流程](/img/treeland/main.drawio.svg)

主代码如下：
```c++
    QWLog::init();
    WServer::initializeQPA();

    QGuiApplication::setAttribute(Qt::AA_UseOpenGLES);
    QGuiApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);
    QGuiApplication::setQuitOnLastWindowClosed(false);
    QGuiApplication app(argc, argv);

    QQmlApplicationEngine waylandEngine;
    QString cursorThemeName = getenv("XCURSOR_THEME");
    waylandEngine.rootContext()->setContextProperty("cursorThemeName", cursorThemeName);

#if QT_VERSION >= QT_VERSION_CHECK(6, 5, 0)
    waylandEngine.loadFromModule("Tinywl", "Main");
#else
    waylandEngine.load(QUrl(u"qrc:/Tinywl/Main.qml"_qs));
#endif
    WServer *server = waylandEngine.rootObjects().first()->findChild<WServer*>();
    Q_ASSERT(server);
    Q_ASSERT(server->isRunning());

    auto backend = server->findChild<WQuickBackend*>();
    Q_ASSERT(backend);
```