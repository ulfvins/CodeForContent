+++
author = "Richard Ulfvin"
categories = ["Hugo"]
tags = ["Codes"]
date = "2018-05-17"
description = "Tesing diffrent code edit layouts"
linktitle = ""
title = "Testing code layouts"
type = "post"

+++

## Introduction
Text goes here...

### HTML
{{< highlight html >}}
<section id="main">
  <div>
    <h1 id="title">{{ .Title }}</h1>
    {{ range .Data.Pages }}
      {{ .Render "summary"}}
    {{ end }}
  </div>
</section>
{{< /highlight >}}

### CSharp
{{< highlight csharp >}}
    using system.test;

    public class test{
        public string test {get; set;}
    }
    var test = new object()
{{< /highlight >}}

### PowerShell
{{< highlight powershell >}}
    $powershell = "test code"
{{< /highlight >}}