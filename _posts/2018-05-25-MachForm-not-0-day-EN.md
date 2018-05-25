---
title: "MachForm: Ceci n'est pas un Zero-Day"
excerpt: "Full disclosure of Not-Zero-Day vulnerabilities in MachForm"
header:
  teaser: "/assets/images/machform-not-0-day/teaser.png"
tags:
  - EN
  - Machform
  - SQL Injection
  - Path Traversal
  - RCE
hidden: true
gallery1:
          - url: /assets/images/Perl-from-EXE/referenced.png
            image_path: /assets/images/Perl-from-EXE/referenced.png
            alt: "Referenced strings"
            title: "Referenced strings"
          - url: /assets/images/Perl-from-EXE/strings_2.png
            image_path: /assets/images/Perl-from-EXE/strings_2.png
            alt: "Strings"
            title: "Strings"
---

Years ago, I discovered some vulnerabilities in MachForm from Appnitro, reported it to the vendor who aknowledged it, issued a fix and even published the notice for the users to update ASAP.

The form creation platform MachForm from Appnitro is subject to SQL injections that lead to path traversal and arbitrary file upload.
The application is widely deployed and with some google dorks it's possible to find various webpages storing sensitive data as Credit Card numbers with corresponding Security Codes.
Otherwise the arbitrary file upload can let an attacker get control of the server by uploading a WebShell.