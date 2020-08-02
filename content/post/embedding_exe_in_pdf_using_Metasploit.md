+++
author = "Sid"
comments = true
date = "2017-11-05T13:59:51Z"
draft = false
image = ""
menu = ""
share = true
tags = ["Metasploit", "Kali","Pentest","Social Engineering", "Adobe PDF EXE"]
title = "Embedding EXE payload in PDF using Metasploit - fixing PDF template errors"

+++

If you have ever tried embedding EXEs in an existing PDF file using Metasploit, most likely you might have come across the error:

`[-] Sorry, I'm picky. Incompatible PDF structure: key not found: "Root". Please try a different PDF template.`

![Load PDF](/images/pdf_before.png)

<!--more-->

Details on the 'Adobe PDF Embedded EXE Social Engineering' Metasploit module can be found [here](https://www.rapid7.com/db/modules/exploit/windows/fileformat/adobe_pdf_embedded_exe).

This is quite an old exploit but still useful in security awareness demonstrations. Recently I had to do one and ran into errors embedding an EXE payload.

To solve the error, open the clean PDF file in Microsoft Word 2013/2016 and again save the file in PDF format.

![Save PDF Using MS Word](/images/save_as_pdf.png)

Load the EXE into the modified PDF and it works !!

![Load PDF](/images/pdf_after.png)

