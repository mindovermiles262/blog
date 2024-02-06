---
title: Activating Windows 10 inside a Virtual Machine
date: 2018-06-07
layout: post
author: andrew
categories: [ windows, "system administration" ]
---

I really tried. I gave Windows 10 a solid month of use before deciding that I couldn't keep using it. It was a tough decision because I use Windows so much for my work. I needed to install Windows 10 inside of a Virtual Machine.

After installing everything and getting it up and running, I was having troubles activating the installation of Windows 10 from our KVM Server at work. After searching the internet, I came across [this article from TechRepublic](https://www.techrepublic.com/article/how-to-install-windows-10-in-a-vm-on-a-linux-machine/) which provided a working solution.

KVM Servers work on a hardware signature which activates the Windows installation per hardware device. By using a Virtual Machine the signature had changed and was invalid for activating windows. The solution? Upload your signature into the VM and activate.

## How To

- From the linux host machine, install `acpica-tools`

- Once installed, run `sudo acpidump -n MSDM`

- In the output, look for the 25-Digit Alpha-numeric signature spanning several lines on the rightmost of the output. This is the product key you need to activate Windows.

- Inside of the Windows VM, Start > Search for "Activation"

- Click on "Change Product Key" and input the signature key you obtained from the host computer. You should now be able to activate Windows 10.