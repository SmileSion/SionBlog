+++
title = "{{ replace .Name "-" " " | title }}"
date = {{ .Date }}
draft = true
tags = ["碎碎念"]
+++

{{< posts/thought title="这里是标题" mood="lake" meta="碎碎念 / {{ (time .Date).Format "01" }}" >}}

这里是导语段落（第一段会自动变大）。

在此继续书写你的感悟...

{{< img src="image-name.jpg" caption="图片描述" alt="配图说明" >}}

> “这里放置你想引用的金句或内心独白。”

---

**这里是最后的强调内容。**

{{< /posts/thought >}}