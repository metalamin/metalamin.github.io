---
title: "Quick Chat(WordPress) - Multiple SQL Injections"
excerpt: "Full details of the Multiple SQL injections in 'Quick Chat' plugin for WordPress"
header:
  teaser: "/assets/images/Perl-from-EXE/teaser.png"
tags:
  - EN
  - WordPress
  - Plugin
  - SQL
gallerydork:
          - url: /assets/images/Quick-Chat-SQLi/Fofa.png
            image_path: /assets/images/Quick-Chat-SQLi/Fofa.png
            alt: "FOFA Dork"
            title: "FOFA Dork"
          - url: /assets/images/Quick-Chat-SQLi/Shodan.png
            image_path: /assets/images/Quick-Chat-SQLi/Shodan.png
            alt: "Shodan Dork"
            title: "Shodan Dork"
gallerypoc1:
          - url: /assets/images/Quick-Chat-SQLi/SQL1_true_req.png
            image_path: /assets/images/Quick-Chat-SQLi/SQL1_true_req.png
            alt: "SQLi True statment request"
            title: "SQLi True statment request"
          - url: /assets/images/Quick-Chat-SQLi/SQL1_true_rsp.png
            image_path: /assets/images/Quick-Chat-SQLi/SQL1_true_rsp.png
            alt: "SQLi True statment response"
            title: "SQLi True statment response"
gallerypoc2:
          - url: /assets/images/Quick-Chat-SQLi/SQL1_false_req.png
            image_path: /assets/images/Quick-Chat-SQLi/SQL1_false_req.png
            alt: "SQLi False statment request"
            title: "SQLi False statment request"
          - url: /assets/images/Quick-Chat-SQLi/SQL1_false_rsp.png
            image_path: /assets/images/Quick-Chat-SQLi/SQL1_false_rsp.png
            alt: "SQLi False statment response"
            title: "SQLi False statment response"
gallerypoc3:
          - url: /assets/images/Quick-Chat-SQLi/SQL2_error_req.png
            image_path: /assets/images/Quick-Chat-SQLi/SQL2_error_req.png
            alt: "SQLi Error based request"
            title: "SQLi Error based request"
          - url: /assets/images/Quick-Chat-SQLi/SQL2_error_rsp.png
            image_path: /assets/images/Quick-Chat-SQLi/SQL2_error_rsp.png
            alt: "SQLi Error based response"
            title: "SQLi Error based response"
            
            
hidden: true
toc: true
toc_label: "Index"
toc_icon: "cog"
---

## Summary
This story starts with a headhunter offering me a job for a major company. No need to say I always screen the employer as I expect they would do to me. If we add the fact that I was a little bored, I ended up finding a 0-day in a wordpress plugin used in one of their servers.

Even if this is not a security compamy they take it very seriously. Kudos for their security team as they managed it blaseling fast.

Let's take a look at this vulnerability found on the plugin ['Quick Chat'](https://wordpress.org/plugins/quick-chat/) for WordPress.


## SQL Injection 1

**Status: still not patched**

The plugin is subject to SQL injections throught the ajax call **quick-chat-ajax-username-check**. As we may apreciate in the code:
<figure class="align-center">
  <img class="align-center" style="width: 75%" src="{{ site.url }}{{ site.baseurl }}/assets/images/Quick-Chat-SQLi/like_escape.png" alt="">
  <figcaption style="text-align: center">Vulnerable code</figcaption>
</figure>

The function *like_escape()* is not meant to act as security measure against SQL injections. In fact it only escapes special characters related to the LIKE statement.( **%** and **_** ). Even the newer **wpdb::esc_like** is not safe as stated on the oficial documentation:  ["The output is not SQL safe."](https://developer.wordpress.org/reference/classes/wpdb/esc_like/)

The vulnerable parameter is **username_check** as we can apreciate on the following POC where the SQL injection is *Blind Boolean Based*.

{% include gallery id="gallerypoc1" %}
{% include gallery id="gallerypoc2" %}

*Note: if "no_participation" is set to 1, login is requiered to preceed with the injection.* 

## SQL Injection 2

**Status: patched on version 4.0**

The plugin was subject to SQL injections throught the ajax call. This SQLi can be found on the **to_delete_ids** parameter when using the action **quick-chat-ajax-delete**.

Proof of concept to get the current database name using an error based technique:
```sql
action=quick-chat-ajax-delete&to_delete_ids[]=666,(select 1 from(select count(*),concat((select (select concat(0x7e,0x27,Hex(cast(database() as char)),0x27,0x7e)) from information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a)
```

{% include gallery id="gallerypoc2" %}

## Dorks
The pluging sets the cookie "quick_chat_alias" so it can be easely tracked searching for it on [shodan.io](https://www.shodan.io/) or [fofa.so](https://fofa.so)

<figure class="align-center">
  <img class="align-center" style="width: 75%" src="{{ site.url }}{{ site.baseurl }}/assets/images/Quick-Chat-SQLi/Fofa.png" alt="">
  <figcaption style="text-align: center">FOFA Dork</figcaption>
</figure>




