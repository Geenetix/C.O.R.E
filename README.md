# C.O.R.E

Core ist die Backend API, welche auf dem CraftLeben.de Netzwerk eingesetzt wird. Dies ist nur die Dokumentation und sollte nur für dich interessant sein, wenn du auf dem Netzwerk als Entwickler tätig bist.

Die API kann sowohl auf Spigot wie auch auf BungeeCord eingesetzt werden.

Es wird viel mit Callbacks gearbeitet. Runnables und Callbacks kann man einfach durch Lambda ersetzen. Dies ist ab Java 8 möglich.
So wird der Code:

```java
     Database.getResult("SELECT * FROM Bans", new DatabaseCallback() {
            @Override
            public void onDataReceived(ResultSet resultSet) throws SQLException {
            }
        });
```
zu
```java
       Database.getResult("SELECT * FROM Bans", resultSet -> {
        });

```
Dies wird in jedem Beispiel angewendet.


## Basics
Die Hautpklassen sind CoreAPI.java und CoreAPIBungee.java. Bitte beachte, dass auf einem Spigot Server nur die CoreAPI.java und auf einem BungeeCord nur die CoreAPIBungee.java geladen wird. Falls eine Instanz von einer der beiden benötigt wird, beispielsweise um einen Scheduler zu registrieren, geht das folgendermaßen:
 ```java       
 CoreAPI.getInstance();
 CoreAPIBungee.getInstance();
 ```
 
 ### Alle Events mit einer Zeile Code registrieren
  ```java
 CoreAPI.registerListener(FFA.getClass(), "de.craftleben.ffa.listener"); 
  ```
### Einen Codeblock asynchron ausführen
  ```java
   CoreAPI.async(new Runnable() {
            @Override
            public void run() {
                
            }
        });
  ```
 
 ## Commands
 Für Spigot Commands ist eine alternative Form zur registrierung von Commands gegeben. Diese Commands können nicht über die Console ausgeführt werden.
 Beispiel:
 
 Erstelle die Klasse FlyCommand.java, diese implementiert CorePlayerCommand. Der Rest ist selbsterklärend. 
onPlayerCommand = Der Code der ausgeführt wird
getCommands = Alle Befehle und alternativen Befehle für den Command
neededRank = Der mindestens erforderliche Rang, um die Befehl auszuführen
getDescription = Beschreibung des Commands z.B. für einen Hilfe Befehl



  ```java
public class FlyCommand implements CorePlayerCommand {

    @Override
    public void onPlayerCommand(Player player, String[] args) {
        if(player.getAllowFlight()){
            player.setAllowFlight(false);
        }else{
            player.setAllowFlight(true);
        }

        player.sendMessage("§eFliegen ist nun§7: §b" + (player.getAllowFlight() ? "aktiviert" : "deaktiviert"));
    }

    @Override
    public List<String> getCommands() {
        return Arrays.asList(
                "fly", "fliegen"
        );
    }

    @Override
    public Rank neededRank() {
        return Rank.MODERATOR;
    }

    @Override
    public String getDescription() {
        return "Aktiviert oder Deaktivert fliegen";
    }
}
 
  ```
Danach muss der Befehl über die CoreAPI registriert werden:
```java
CoreAPI.registerCommand(new FlyCommand());
```

##Datenbank
Für alle Datenbankanfragen wird die Klasse Database.java verwendet.

### Ein ResultSet anfordert - asynchron mit Callback
Die Methode getResult gibt ein DatabaseCallback mit einer Runnable zurück.
```java
        Database.getResult("SELECT * FROM Bans", resultSet -> {
            while(resultSet.next()){
                UUID uuid = UUID.fromString(resultSet.getString("uuid"));
            }
        });
```

### Ein Update senden - Asynchron
```java
  Database.update("INSERT INTO Bans (uuid, banneduntil) VALUES (1234-abcdef-de5678-fghij, 15678861251)");
```

## Inventories und ItemStacks
Die API beinhaltet sowohl einen ItemBuilder sowie einen InventoryBuilder, mit denen es deutlich einfacher ist, z.B. Klickbare Inventare zu  erstellen:

withBackground() = Füllt das ganze Inventar mit z.B. Glassplatten, ohne Namen.
interactionDisabled() = Verhindert, dass Items herausgenommen werden können
openToPlayer() = Öffnet das fertige Inventar einem Spieler

Beispiel Kompass:
```java
    public static void createCompass(Player player){
        String title = "§cKompass";
        Integer size = 9*3;

        InventoryBuilder inventoryBuilder = new InventoryBuilder(title, size);
        inventoryBuilder.withBackground(new ItemBuilder(Material.STAINED_GLASS_PANE, 1, 14));
        inventoryBuilder.interactionDisabled();

        ItemBuilder spawn = new ItemBuilder(Material.NETHER_STAR)
                .withName("§bSpawn")
                .withLore("§7Klicke zum teleportieren")
                .onLeftClick((clickPlayer, event) -> {
                    Location spawnLocation = Bukkit.getWorld("FarmWelt").getSpawnLocation();
                    clickPlayer.teleport(spawnLocation);
                });
        inventoryBuilder.addItem(spawn, 13);

        inventoryBuilder.openToPlayer(player);
    }
```



```java

```


 


 
 

 
