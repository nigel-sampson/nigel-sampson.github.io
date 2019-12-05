---
layout: post
title: Nord theme for Windows Terminal
---

I've been playing around with [Windows Terminal][terminal] for the last few weeks and been really impressed.  In prevoius terminals and in VS Code lately I've had the [Nord theme][nord] which while totally subjective looks pretty cool.  

It has [ports][ports] for a lot of applications, but not Windows Terminals. Thankfully Anais Betts put together an awesome point on how to [convert a VS Code theme to a Windows Terminal][vs-code].

For anyone else who's keen there it is.

``` json
{
    "name" : "nord",
    "background" : "#2e3440",
    "foreground" : "#d8dee9",
    "black": "#3b4252",
    "blue": "#81a1c1",
    "brightBlack": "#4c566a",
    "brightBlue": "#81a1c1",
    "brightCyan": "#8fbcbb",
    "brightGreen": "#a3be8c",
    "brightPurple": "#b48ead",
    "brightRed": "#bf616a",
    "brightWhite": "#eceff4",
    "brightYellow": "#ebcb8b",
    "cyan": "#88c0d0",
    "green": "#a3be8c",
    "purple": "#b48ead",
    "red": "#bf616a",
    "white": "#e5e9f0",
    "yellow": "#ebcb8b"
}
```

[nord]: https://www.nordtheme.com/
[ports]: https://www.nordtheme.com/ports
[vs-code]: https://blog.anaisbetts.org/vs-code-themes-in-windows-terminal/
[terminal]: https://github.com/microsoft/terminal
