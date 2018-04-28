---
layout: post
title: 在 awesome wm 下使用两个显示器（续）
tags:
  - awesome
---

前几天设置两个显示器，还剩下一个问题：当只使用一个 screen 的时候，切换 screen 的时候，所有窗口和 systray
自动发送到另一个 screen 去，且保持布局不变。

在解决这个问题之前，我只能同时使用两个显示器，并且左右布局，各不相干，当使用外接显示器的时候，
先将其切换成 primary，这样 systray 会到外接显示器上；而当要使用笔记本（开会的时候）的时候，
则反之。

但是，显然，所有窗口如果想要带走，那么还需要用 `modkey + o`
一个一个发送到笔记本上，还是挺不方便的。

而现在，解决了切换 screen，所有窗口和 systray 都自动保持原样，发送到另一个
screen 之后，现在就畅快多了，要带走笔记本的时候，只需要切换为只使用笔记本显示器模式即可。

而当要使用外接显示器的时候，则切换回只使用外接显示器即可。

切换显示器依然使用
[上一篇博客]({% post_url 2018-04-24-awesome-multi-monitor-1 %})
介绍的 xrandr.lua，而保持所有 window 和 tag
布局则使用了 reddit 上的一个方法，稍微修改代码，删除显示器分辨率判断语句即可。

修改后如下：

```
tag.connect_signal("request::screen", function(t)
    for s in screen do
        if s ~= t.screen then
            local t2 = awful.tag.find_by_name(s, t.name)
            if t2 then
                t:swap(t2)
            else
                t.screen = s
            end
            return
        end
    end
end)
```

把上面这几行加入 awesome 的配置文件 rc.lua 中，重启 awesome 即可。
