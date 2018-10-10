---
layout: post
title:  "代理模式(Proxy)瞎扯"
categories: 学习笔记
tags: java Proxy
excerpt: C#的代理模式的示例，这篇是摘录的
---

当我们需要使用的对象很复杂或者需要很长时间去构造，这时就可以使用代理模式(Proxy)。例如：如果构建一个对象很耗费时间和计算机资源，代理模式(Proxy)允许我们控制这种情况，直到我们需要使用实际的对象。一个代理(Proxy)通常包含和将要使用的对象同样的方法，一旦开始使用这个对象，这些方法将通过代理(Proxy)传递给实际的对象。 
  
一些可以使用代理模式(Proxy)的情况：

1. 一个对象，比如一幅很大的图像，需要载入的时间很长。
2. 一个需要很长时间才可以完成的计算结果，并且需要在它计算过程中显示中间结果
3. 一个存在于远程计算机上的对象，需要通过网络载入这个远程对象则需要很长时间，特别是在网络传输高峰期。
4. 一个对象只有有限的访问权限，代理模式(Proxy)可以验证用户的权限

代理模式(Proxy)也可以被用来区别一个对象实例的请求和实际的访问，例如：在程序初始化过程中可能建立多个对象，但并不都是马上使用，代理模式(Proxy)可以载入需要的真正的对象。

这是一个需要载入和显示一幅很大的图像的程序，当程序启动时，就必须确定要显示的图像，但是实际的图像只能在完全载入后才可以显示！这时我们就可以使用代理模式(Proxy)。这个代理模式(Proxy)可以延迟实际图像的载入，直到它接收到一个paint请求。在实际图像的载入期间我们可以通过代理模式(Proxy)在实际图像要显示的位置预先载入一个比较小、简单的图形。

图像Proxy代码：

```
    Public Class ImageProxy

    Private done As Boolean

    Private tm As Timer

    Public Sub New()

        done = False

        '设置timer 延迟5秒

        tm = New Timer( _

        New TimerCallback(AddressOf tCallback), Me, 5000, 0)

    End Sub

    Public Function isReady() As Boolean

Return done

    End Function

   Public Function getImage() As Image

        Dim img As Imager

        '显示预先的图像，直到实际图像载入完成

        If isReady Then

            img = New FinalImage()

        Else

            img = New QuickImage()

        End If

        Return img.getImage

    End Function

    Public Sub tCallback(ByVal obj As Object)

        done = True

        tm.Dispose()

    End Sub

  End Class
```

  定义一个简单的接口：

  
```
Public Interface Imager

    Function getImage() As image

  End Interface
```

  实现接口：

  预先载入的图像的类：


```
Public Class QuickImage

    Implements Imager

Public Function getImage() As Image _

            Implements Imager.getImage

        Return New bitmap("Box.gif")

    End Function

  End Class
```

  载入实际图像的类：

  
```
Public Class FinalImage

    Implements Imager

    Public Function getImage() As Image _

        Implements Imager.getImage

        Return New Bitmap("flowrtree.jpg")

    End Function

  End Class
```

  在显示图像的窗体中,定义一个图像代理的(Proxy)实例，在载入图像按钮事件中，载入图像：

  
```
Private imgProxy As ImageProxy

    Public Sub New()

        MyBase.New

        Form1 = Me

        InitializeComponent

        imgproxy = New ImageProxy()

    End Sub

    Protected Sub btLoad_Click(ByVal sender As Object, ByVal e As System.EventArgs) Handles btLoad.Click

        pic.Image = imgProxy.getImage

    End Sub
```

**总结：**

这只是一个很简单的例子（例子来自于《c#设计模式》），通过这个例子可以对代理(Proxy)有初步的认识！Adapter模式和代理模式(Proxy)都是在对象间构造一个简单的层。然而，Adapter模式向对象提供一个不同的接口，代理模式(Proxy)为对象提供相同的接口
