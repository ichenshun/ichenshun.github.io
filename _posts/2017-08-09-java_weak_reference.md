---
layout: post
title: WeakReference包装Listener对象
categories: [Java]
keywords: WeakReference, Java
---

被观察对象使用WeakReference包装 Listener对象的原因是，当Listener对象不再被使用时，可以不用通知被观察对象移除Listener对象，WeakReference会自动释放它包装的Listener对象。

当很难判断Listener对象什么时候不再被使用，或者很难给被观察对象发送通知时，使用上面的技术可以很好的绕过这些问题，让代码变得更简洁。