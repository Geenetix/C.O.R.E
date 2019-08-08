# C.O.R.E

Core ist die Backend API, welche auf dem CraftLeben.de Netzwerk eingesetzt wird. Dies ist nur die Dokumentation und sollte nur für dich interessant sein, wenn du auf dem Netzwerk als Entwickler tätig bist.

Die API kann sowohl auf Spigot wie auch auf BungeeCord eingesetzt werden.

## Basics
Die Hautpklassen sind CoreAPI.java und CoreAPIBungee.java. Falls eine Instanz von einer der beiden benötigt wird, beispielsweise um einen Scheduler zu registrieren, geht das folgendermaßen:
 ```java       
 CoreAPI.getInstance();
 CoreAPIBungee.getInstance();
 ```
