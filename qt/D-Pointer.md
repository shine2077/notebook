qtmaterialoverlaywidget.h

```c++
#ifndef QTMATERIALOVERLAYWIDGET_H
#define QTMATERIALOVERLAYWIDGET_H

#include <QtWidgets/QWidget>

class QtMaterialOverlayWidget : public QWidget
{
    Q_OBJECT

public:
    explicit QtMaterialOverlayWidget(QWidget *parent = 0);
    ~QtMaterialOverlayWidget();

protected:
    bool event(QEvent *event) Q_DECL_OVERRIDE;
    bool eventFilter(QObject *obj, QEvent *event) Q_DECL_OVERRIDE;

    virtual QRect overlayGeometry() const;

private:
    Q_DISABLE_COPY(QtMaterialOverlayWidget)
};

#endif // QTMATERIALOVERLAYWIDGET_H
```

qtmaterialdialog.h

```c++
#ifndef QTMATERIALDIALOG_H
#define QTMATERIALDIALOG_H

#include <QScopedPointer>
#include "qtmaterialoverlaywidget.h"

class QLayout;
class QtMaterialDialogPrivate;

class QtMaterialDialog : public QtMaterialOverlayWidget
{
    Q_OBJECT

public:
    explicit QtMaterialDialog(QWidget *parent = 0);
    ~QtMaterialDialog();

    QLayout *windowLayout() const;
    void setWindowLayout(QLayout *layout);

public slots:
    void showDialog();
    void hideDialog();

protected:
    void paintEvent(QPaintEvent *event) Q_DECL_OVERRIDE;

    const QScopedPointer<QtMaterialDialogPrivate> d_ptr;

private:
    Q_DISABLE_COPY(QtMaterialDialog)
    Q_DECLARE_PRIVATE(QtMaterialDialog)
};

#endif // QTMATERIALDIALOG_H
```

qtmaterialdialog_p.h

```c++
#ifndef QTMATERIALDIALOG_P_H
#define QTMATERIALDIALOG_P_H

#include <QtGlobal>

class QStateMachine;
class QtMaterialDialog;
class QStackedLayout;
class QtMaterialDialogWindow;
class QtMaterialDialogProxy;

class QtMaterialDialogPrivate
{
    Q_DISABLE_COPY(QtMaterialDialogPrivate)
    Q_DECLARE_PUBLIC(QtMaterialDialog)

public:
    QtMaterialDialogPrivate(QtMaterialDialog *q);
    ~QtMaterialDialogPrivate();

    void init();

    QtMaterialDialog       *const q_ptr;
    QtMaterialDialogWindow *dialogWindow;
    QStackedLayout         *proxyStack;
    QStateMachine          *stateMachine;
    QtMaterialDialogProxy  *proxy;
};

#endif // QTMATERIALDIALOG_P_H
```

qtmaterialdialog.cpp

```c++
#include "qtmaterialdialog.h"
#include "qtmaterialdialog_p.h"
```


