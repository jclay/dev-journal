# Connecting a per frame callback in Qt3D

With Qt3D, we can use signals and slots in order to connect a per-frame callback.

In short, we setup a `QFrameAction` on our root entity. We set up a handler and a slot to be called when the signal is emitted.
We can use this to handle lower level functions in the graphcis pipeline such as manually updating the data in a vertex buffer.

## Example Implementation

```cpp
# BufferSyncHandler.h
class BufferSyncHandler: public QObject {
public:
	BufferSyncHandler();
	~BufferSyncHandler();
public slots:
	void SyncPositions(float dt)
};
```

```cpp
# BufferSyncHandler.cpp
#include "BufferSyncHandler.h"
#include <iostream>
#include <stdio.h>
using namespace std;

BufferSyncHandler::BufferSyncHandler() { }
BufferSyncHandler::~BufferSyncHandler() { }

void BufferSyncHandler::SyncPositions(float dt) { cout << "Time elapsed since last call" << dt << endl; }

```

The following boiler plate code is adapted from the Qt3D `basicshapes` example. Note that I have connected the QFrameAction `triggered` signal to our handler.

```cpp
#include "scenemodifier.h"

#include <QGuiApplication>

#include <Qt3DRender/qcamera.h>
#include <Qt3DCore/qentity.h>
#include <Qt3DRender/qcameralens.h>

#include <QtWidgets/QApplication>
#include <QtWidgets/QWidget>
#include <QtWidgets/QHBoxLayout>
#include <QtWidgets/QCheckBox>
#include <QtWidgets/QCommandLinkButton>
#include <QtGui/QScreen>

#include <Qt3DInput/QInputAspect>

#include <Qt3DExtras/qtorusmesh.h>
#include <Qt3DRender/qmesh.h>
#include <Qt3DRender/qtechnique.h>
#include <Qt3DRender/qmaterial.h>
#include <Qt3DRender/qeffect.h>
#include <Qt3DRender/qtexture.h>
#include <Qt3DRender/qrenderpass.h>
#include <Qt3DRender/qsceneloader.h>
#include <Qt3DRender/qpointlight.h>
#include <Qt3DLogic/qframeaction.h>

#include <Qt3DCore/qtransform.h>
#include <Qt3DCore/qaspectengine.h>

#include <Qt3DRender/qrenderaspect.h>
#include <Qt3DExtras/qforwardrenderer.h>

#include <Qt3DExtras/qt3dwindow.h>
#include <Qt3DExtras/qfirstpersoncameracontroller.h>

#include "MassBufferSyncHandler.h"

int main(int argc, char **argv)
{
    QApplication app(argc, argv);
    Qt3DExtras::Qt3DWindow *view = new Qt3DExtras::Qt3DWindow();
    view->defaultFrameGraph()->setClearColor(QColor(QRgb(0x4d4d4f)));
    QWidget *container = QWidget::createWindowContainer(view);
    QSize screenSize = view->screen()->size();
    container->setMinimumSize(QSize(200, 100));
    container->setMaximumSize(screenSize);

    QWidget *widget = new QWidget;
    QHBoxLayout *hLayout = new QHBoxLayout(widget);
    QVBoxLayout *vLayout = new QVBoxLayout();
    vLayout->setAlignment(Qt::AlignTop);
    hLayout->addLayout(vLayout);
    hLayout->addWidget(container, 1);

    widget->setWindowTitle(QStringLiteral("Basic shapes"));

    Qt3DInput::QInputAspect *input = new Qt3DInput::QInputAspect;
    view->registerAspect(input);

    // Root entity
    Qt3DCore::QEntity *rootEntity = new Qt3DCore::QEntity();

    // Camera
    Qt3DRender::QCamera *cameraEntity = view->camera();

    cameraEntity->lens()->setPerspectiveProjection(45.0f, 16.0f / 9.0f, 0.1f, 1000.0f);
    cameraEntity->setPosition(QVector3D(0, 0.0f, 20.0f));
    cameraEntity->setUpVector(QVector3D(0, 1.0f, 0.0f));
    cameraEntity->setViewCenter(QVector3D(0, 0, 0));

    Qt3DCore::QEntity *lightEntity = new Qt3DCore::QEntity(rootEntity);
    Qt3DRender::QPointLight *light = new Qt3DRender::QPointLight(lightEntity);
    light->setColor("white");
    light->setIntensity(1);
    lightEntity->addComponent(light);
    Qt3DCore::QTransform *lightTransform = new Qt3DCore::QTransform(lightEntity);
    lightTransform->setTranslation(cameraEntity->position());
    lightEntity->addComponent(lightTransform);

    // For camera controls
    Qt3DExtras::QFirstPersonCameraController *camController = new Qt3DExtras::QFirstPersonCameraController(rootEntity);
    camController->setCamera(cameraEntity);

    // Scenemodifier
    SceneModifier *modifier = new SceneModifier(rootEntity);

    // Sync handler
    BufferSyncHandler *mbSync = new BufferSyncHandler();

    // Create a frame action and set it to our root entity
    Qt3DLogic::QFrameAction *fa = new Qt3DLogic::QFrameAction;
    rootEntity->addComponent(fa);
    // Connect the QFrameAction triggered signal to the slot on our handler.
    fa->connect(fa, &Qt3DLogic::QFrameAction::triggered, mbSync, &MassBufferSyncHandler::SyncMassPositions);

    // Set root object of the scene
    view->setRootEntity(rootEntity);

    // Create control widgets
    QCommandLinkButton *info = new QCommandLinkButton();
    info->setText(QStringLiteral("Qt3D ready-made meshes"));
    info->setDescription(QString::fromLatin1("Qt3D provides several ready-made meshes, like torus, cylinder, cone, "
        "cube, plane and sphere."));
    info->setIconSize(QSize(0, 0));


    QCheckBox *cuboidCB = new QCheckBox(widget);
    cuboidCB->setChecked(true);
    cuboidCB->setText(QStringLiteral("Cuboid"));


    vLayout->addWidget(cuboidCB);

    QObject::connect(cuboidCB, &QCheckBox::stateChanged,
        modifier, &SceneModifier::enableCuboid);

    cuboidCB->setChecked(true);

    // Show window
    widget->show();
    widget->resize(1200, 800);

    return app.exec();
}


```

## Further Reading
- https://forum.qt.io/topic/85017/qt3d-synchronize-with-frame-rate
- https://doc.qt.io/qt-5/qt3dlogic-qframeaction.html#triggered
- https://doc.qt.io/qt-5/signalsandslots.html
