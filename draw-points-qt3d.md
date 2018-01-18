# Drawing points with Qt3D
Qt3D provides a nice wrapper around many of the platforms that run GL based platforms. It provides a nice framegraph and entity system allowing
for easy management of our assets and meshes. 

While Qt3D handles this complexity nicely, they don't provide an out of the box way to draw simple primitives such as lines or points. Based on digging through the Qt3D source code, I was able to come up with the following solution.
The `QPlaneMesh`, `QCuboidMesh` and their corresponding gemoetry implementations are particularly heplful. I have linked to those in the Qt3D source below.

## Implementing 
We ultimately need to create our own Geometry and tell it to use use `Points` as its primitive. With a few Qt3D helpers, we are ultimately filling our own OpenGL buffer, setting up our shaders, and telling the vertex attribute
how to read the data from our buffer.

```cpp
// ============================
// Point.h
// ============================

#include <Qt3DExtras/qt3dextras_global.h>
#include <Qt3DRender/qbuffer.h>
#include <Qt3DRender/qgeometryrenderer.h>
#include <qvector3d.h>
#include <QtCore/qsize.h>

using namespace Qt3DRender;

class Point : public QGeometryRenderer {
Q_OBJECT
public:
    explicit Point(Qt3DCore::QNode *parent = nullptr);
    ~Point();
public slots:
    void setPosition(QVector3D position);
};

class PointGeometry : public QGeometry {
Q_OBJECT
public:
    explicit PointGeometry(QNode *parent = nullptr);
    ~PointGeometry();


    QVector3D position;

    void init();
    void updateVertices();
    void updateIndices();

    QBuffer * vertexBuffer;
    QBuffer * indexBuffer;
    QAttribute * positionAttribute;
    QAttribute * indexAttribute;

public slots:
    void setPosition(QVector3D position) ;

};

```

And now for our implementation
```cpp
// ============================
// Point.cpp
// ============================
#include <Qt3DRender/qattribute.h>
#include <Qt3DRender/qbuffer.h>
#include <Qt3DRender/qpointsize.h>
#include <Qt3DRender/qbufferdatagenerator.h>
#include <limits>
#include "Point.h"

using namespace Qt3DRender;

Point::Point(QNode *parent) : QGeometryRenderer(parent) {
    PointGeometry *geometry = new PointGeometry(this);
    // tell OpenGL to expect a point primtiive
    // https://www.khronos.org/opengl/wiki/Primitive#Point_primitives
    setPrimitiveType(PrimitiveType::Points);
    QGeometryRenderer::setGeometry(geometry);
}

Point::~Point() { }

void Point::setPosition(QVector3D position) {
    static_cast<PointGeometry *>(geometry())->setPosition(position);
}

// set up a vertex buffer with our single vec3 position
QByteArray createPointVertexData(QVector3D position) {
    const int nVerts = 1;
    const quint32 stride = 3 * sizeof(float);
    // create  a new buffer
    QByteArray bufferBytes;
    // set buffer size to number of vertices
    // we're only drawing one point, so we only have one
    // vec3 representing the position of our point
    bufferBytes.resize(stride * nVerts);

    // tell c++ to interpret this buffer as float so
    // we can use the next few lines to fill it 
    float* fptr = reinterpret_cast<float*>(bufferBytes.data());

    *fptr++ = position.x();
    *fptr++ = position.y();
    *fptr++ = position.z();

    return bufferBytes;
}

// set up an index buffer telling OpenGL
// to render our single point
QByteArray createPointIndexData() {
    QByteArray indexBytes;
    
    indexBytes.resize(1 * sizeof(quint16));
    quint16* indexPtr = reinterpret_cast<quint16*>(indexBytes.data());

    *indexPtr++ = 0;

    return indexBytes;
}

// this functor is responsible for updating the vertex buffer
// it is set as our data generator below
class PointVertexBufferFunctor : public QBufferDataGenerator {
public:
    explicit PointVertexBufferFunctor(QVector3D position) : position(position) {}

    ~PointVertexBufferFunctor() {}

    QByteArray operator()() Q_DECL_FINAL {
        return createPointVertexData(position);
    }

    bool operator ==(const QBufferDataGenerator &other) const Q_DECL_FINAL {
        const PointVertexBufferFunctor *otherFunctor = functor_cast<PointVertexBufferFunctor>(&other);
        if (otherFunctor != nullptr)
            return (otherFunctor->position == position);
        return false;
    }

    QT3D_FUNCTOR(PointVertexBufferFunctor)

    private:
    QVector3D position;
};

// this functor is responsible for updating the index buffer
// it is set as our data generator below
class PointIndexBufferFunctor : public QBufferDataGenerator
{
public:
    explicit PointIndexBufferFunctor() {}
    ~PointIndexBufferFunctor() {}

    QByteArray operator()() Q_DECL_FINAL
    {
        return createPointIndexData();
    }

    bool operator ==(const QBufferDataGenerator &other) const Q_DECL_FINAL
    {
        const PointIndexBufferFunctor *otherFunctor = functor_cast<PointIndexBufferFunctor>(&other);
        if (otherFunctor != nullptr)
            return (true);
        return false;
    }

    QT3D_FUNCTOR(PointIndexBufferFunctor)

    private:
        QSize resolution;
};

PointGeometry::PointGeometry(PointGeometry::QNode *parent) : QGeometry(parent) {
    this->init();
}

PointGeometry::~PointGeometry() { }

void PointGeometry::setPosition(QVector3D position) {
    this->position = position;
    updateVertices();
}

void PointGeometry::updateVertices() {
    positionAttribute->setCount(3);
    vertexBuffer->setDataGenerator(QSharedPointer<PointVertexBufferFunctor>::create(position));
}

void PointGeometry::updateIndices() {
    indexAttribute->setCount(1);
    indexBuffer->setDataGenerator(QSharedPointer<PointIndexBufferFunctor>::create());
}

void PointGeometry::init() {
    positionAttribute = new QAttribute();
    indexAttribute = new QAttribute();
    vertexBuffer = new Qt3DRender::QBuffer();
    indexBuffer = new Qt3DRender::QBuffer();

    const int nVerts = 3;
    const int stride = 0; // since the vertices are "side by side" in the buffer

    // this ultimately ends up configuring the OpenGL vertex array object
    // https://www.khronos.org/opengl/wiki/Vertex_Specification
    // this tells Qt3D / OpenGL to read the data from the buffer,
    // passing the 3 floats into the gl_Position parameter
    // on our shader (in this case the Qt3D default shader)
    positionAttribute->setName(QAttribute::defaultPositionAttributeName());
    positionAttribute->setVertexBaseType(QAttribute::Float);
    positionAttribute->setVertexSize(3); 
    positionAttribute->setAttributeType(QAttribute::VertexAttribute);
    positionAttribute->setBuffer(vertexBuffer);
    positionAttribute->setByteStride(stride);
    positionAttribute->setCount(nVerts);

    indexAttribute->setAttributeType(QAttribute::IndexAttribute);
    indexAttribute->setVertexBaseType(QAttribute::UnsignedShort);
    indexAttribute->setBuffer(indexBuffer);

    // Each primitive has 3 vertives
    indexAttribute->setCount(1);

    vertexBuffer->setDataGenerator(QSharedPointer<PointVertexBufferFunctor>::create(position));
    indexBuffer->setDataGenerator(QSharedPointer<PointIndexBufferFunctor>::create());

    addAttribute(positionAttribute);
    addAttribute(indexAttribute);
}

```

## Usage

You render the points on screen in your `SceneModifer` class with something similar to the following:

```cpp
void SceneModifier::AddPoints() {
    points_root = new Qt3DCore::QEntity(m_rootEntity);

    for (int i = 0; i < 10; i++) {
        for (int j = 0; j < 10; j++) {
            auto point_entity = new Qt3DCore::QEntity(points_root);
            auto point = new Point();
            auto point_transform = new Qt3DCore::QTransform();
            auto point_material = new Qt3DExtras::QPhongMaterial();
            point->setPosition(QVector3D(i, j, 0));

            // this is my hacky way of setting point size
            // the better way to do this is probably to create
            // your own shader and then use QPointSize::SizeMode::Programmable
            // that's for another journal...
            auto effect = point_material->effect();
            for (auto t : effect->techniques()) {
                for (auto rp : t->renderPasses()) {
                    auto pointSize = new QPointSize();
                    pointSize->setSizeMode(QPointSize::SizeMode::Fixed);
                    pointSize->setValue(4.0f);
                    rp->addRenderState(pointSize);
                }
            }

            point_material->setAmbient(QColor(255, 255, 255));

            point_entity->addComponent(point);
            point_entity->addComponent(point_material);
            point_entity->addComponent(point_transform);

        }
    }
}
```

## Further Reading
- [The source for QPlaneMesh.cpp](https://github.com/qt/qt3d/blob/5.10/src/extras/geometries/qplanemesh.cpp)
- [The source for QPlaneGeometry.cpp](https://github.com/qt/qt3d/blob/5.10/src/extras/geometries/qplanegeometry.cpp)
