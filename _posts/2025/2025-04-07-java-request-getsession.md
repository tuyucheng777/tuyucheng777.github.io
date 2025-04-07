---
layout: post
title:  request.getSession()和request.getSession(true)之间的区别
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在本快速教程中，我们将介绍调用HttpServletRequest#getSession()和HttpServletRequest#getSession(boolean)之间的区别。

## 2. 有什么区别？

getSession()和getSession(boolean)方法非常相似，不过还是有一点区别。区别在于如果会话尚不存在，是否应该创建它。

**调用getSession()和getSession(true)功能相同**：检索当前会话，如果尚不存在，则创建它。

但是，**调用getSession(false)会检索当前会话，如果尚不存在会话，则返回null**。除此之外，当我们想询问会话是否存在时，这非常方便。

## 3. 示例

在此示例中，我们考虑这种场景：

- 用户输入用户ID并登录应用程序
- 然后用户输入username和age，并希望为登录用户更新这些详细信息

我们将用户值存储在会话中，以了解HttpServletRequest#getSession()和HttpServletRequest#getSession(boolean)的用法。

首先，让我们创建一个Servlet，并在其doGet()方法中使用HttpServletRequest#getSession()：

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    HttpSession session = request.getSession();
    session.setAttribute("userId", request.getParameter("userId"));
}
```

此时，Servlet将检索现有会话，如果不存在，则为登录用户创建一个新的会话。

接下来，我们将在会话中设置userName属性。

因为我们想要根据相应的用户ID更新用户的详细信息，所以我们想要相同的会话，而不想创建新的会话来存储用户名。

所以现在，我们将使用带有false值的HttpServletRequest#getSession(boolean)：

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    HttpSession session = request.getSession(false);
    if (session != null) {
        session.setAttribute("userName", request.getParameter("userName"));
    }
}
```

这将导致在先前设置userId的同一会话中设置userName属性。

## 4. 总结

在本教程中，我们解释了HttpServletRequest#getSession()和HttpServletRequest#getSession(boolean)方法之间的区别。