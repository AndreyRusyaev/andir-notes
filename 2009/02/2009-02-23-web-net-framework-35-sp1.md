Неприятности с web-сервисами после установки .Net Framework 3.5 SP1
===================================================================

    published: 2009-02-23 
    tags: .net 
    permalink: https://andir-notes.blogspot.com/2009/02/web-net-framework-35-sp1.html

Внезапно, после установки обновления [.Net Framework 3.5 Service Pack 1](http://www.microsoft.com/downloads/details.aspx?familyid=ab99342f-5d1a-413d-8319-81da479ab0d7&displaylang=en), некоторые веб-сервисы с NTLM-аутентификацией начинают выдавать 401.1 - Unauthorized: Logon Failed.

Происходит это в тех нередких случаях, когда пытаешься вызвать веб-сервис, расположенный на том же IIS, но обращаясь при этом по длинному имени (FQDN). Причина возникновения ошибки кроется в том, что в новом сервис-паке при использовании аутентификации NTLM стали учитывать нововведение в серверной безопасности под названием “Loopback check” ([kb896861](http://support.microsoft.com/kb/896861 "support.microsoft.com: You receive error 401.1 when you browse a Web site that uses Integrated Authentication and is hosted on IIS 5.1 or IIS 6")). Полное описание изменений приведено в MSDN и там же есть способы исправления ситуации (аналогично статье из KB): [Changes to NTLM authentication for HTTPWebRequest in Version 3.5 SP1](http://msdn.microsoft.com/en-us/library/cc982052.aspx "MSDN: Changes to NTLM authentication for HTTPWebRequest in Version 3.5 SP1").

_Примечание: “Loopback check security feature” была добавлена ещё сервис-паками Windows XP Service Pack 2 и Windows 2003 Service Pack 1, но до этого не учитывалась механизмами аутентификации .Net Framework._