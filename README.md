# C.O.R.E - ControlOrganisation and Rapidly-evolving Emulation

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


## Basics<br>
Die Hautpklassen sind CoreAPI.java und CoreAPIBungee.java. Bitte beachte, dass auf einem Spigot Server nur die CoreAPI.java und auf einem BungeeCord nur die CoreAPIBungee.java geladen wird. Falls eine Instanz von einer der beiden benötigt wird, beispielsweise um einen Scheduler zu registrieren, geht das folgendermaßen:
 ```java       
 CoreAPI.getInstance();
 CoreAPIBungee.getInstance();
 ```
 
 ### Alle Events mit einer Zeile Code registrieren<br>
 Du gibst nur die Mainklasse deines Plugins und der Pfad zu deinem Package mit Listenern.
  ```java
 CoreAPI.registerListener(FFA.getClass(), "de.craftleben.ffa.listener"); 
  ```
### Einen Codeblock asynchron ausführen
  ```java
        CoreAPI.async(() -> {
            //ASYNC CODE
        });
  ```
  
 ### Packets senden
 ```java
PacketManager.sendPacket(player, new PacketPlayOutTitle(...));
```

### Prefixes
Jedes System hat einen eigenen Prefix, der von der COREAPI verwaltet wird. Jede Standard Nachricht (z.B. "Die Runde beginnt in ..." oder "... abgeschlossen") wird mit dem Standard Prefix des jeweiligen Systems gesendet (z.B. Prefix.LOBBY). Jede Fehlernachricht oder Syntaxfehlernachricht (z.B. "Du hast zu wenige Coins" oder "Verwende /...") nutzt den Error Prefix des jeweiligen Systems (z.B. Prefix.LOBBY_ERROR)<br>
Beispiel:<br>
 ```java
                player.sendMessage(Prefix.LOBBY + "Wenn du weiter so schlecht codest, zerleg ich dein Motherboard und mach aus dir ein Weinregal");
                player.sendMessage(Prefix.LOBBY_ERROR + "Du codest zu schlecht - Verwende /hub schrauber"); 
```
 
 
 ## Commands<br>
 Für Spigot Commands ist eine alternative Form zur registrierung von Commands gegeben. Diese Commands können nicht über die Console ausgeführt werden. Die Befehle müssen NICHT in der plugin.yml registriert werden!
 Beispiel:
 
 Erstelle die Klasse FlyCommand.java, diese implementiert CorePlayerCommand. Der Rest ist selbsterklärend. <br>
onPlayerCommand = Der Code der ausgeführt wird<br>
getCommands = Alle Befehle und alternativen Befehle für den Command<br>
neededRank = Der mindestens erforderliche Rang, um die Befehl auszuführen<br>
getDescription = Beschreibung des Commands z.B. für einen Hilfe Befehl<br>



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

## Datenbank<br>
Für alle Datenbankanfragen wird die Klasse Database.java verwendet.

### Ein ResultSet anfordern - asynchron mit Callback
Die Methode getResult gibt ein DatabaseCallback mit einer Runnable zurück.
```java
        Database.getResult("SELECT * FROM Bans", resultSet -> {
            while(resultSet.next()){
                UUID uuid = UUID.fromString(resultSet.getString("uuid"));
                ...
            }
        });
```

### Ein Update senden - Asynchron<br>
```java
  Database.update("INSERT INTO Bans (uuid, banneduntil) VALUES (1234-abcdef-de5678-fghij, 15678861251)");
```

## Inventories, ItemStacks & co.<br>
Die API beinhaltet sowohl einen ItemBuilder sowie einen InventoryBuilder, mit denen es deutlich einfacher ist, z.B. Klickbare Inventare zu  erstellen:

### Inventories<br>
withBackground() = Füllt das ganze Inventar mit z.B. Glassplatten, ohne Namen.<br>
disableInteraction() = Verhindert, dass Items herausgenommen werden können<br>
openToPlayer() = Öffnet das fertige Inventar einem Spieler<br>

Beispiel Kompass:
```java
    public static void createCompass(Player player){
        String title = "§cKompass";
        Integer size = 9*3;

        InventoryBuilder inventoryBuilder = new InventoryBuilder(title, size);
        inventoryBuilder.withBackground(new ItemBuilder(Material.STAINED_GLASS_PANE, 1, 14));
        inventoryBuilder.disableInteraction(true);

        ItemBuilder spawn = new ItemBuilder(Material.NETHER_STAR)
                .withName("§bSpawn")
                .withLore("§7Klicke zum teleportieren")
                .onLeftClick((clickPlayer, event) -> {
                    Location spawnLocation = Bukkit.getWorld("world").getSpawnLocation();
                    clickPlayer.teleport(spawnLocation);
                });
        inventoryBuilder.addItem(spawn, 13);

        inventoryBuilder.openToPlayer(player);
    }
```
In Minecraft sieht das dann so aus:<br>
![alt text](https://www.bilder-upload.eu/upload/ecfdf6-1565280963.gif)

### ItemBuilder<br>
Der ItemBuilder ist sehr selbsterklärend. Man erstellt pro Item einen neuen ItemBuilder, welchem man dann nach und nach mehr Informationen angehängt werden. Am Ende wird mit toItemStack() ein ItemStack generiert. Spielerköpfe können mit toPlayerHead() erstellt werden. 
Beispiele:
```java
        ItemStack diamond = new ItemBuilder(Material.DIAMOND)
                .withName("§bDiamond")
                .withLore("§aSELTEN!")
                .withItemFlag(ItemFlag.HIDE_ENCHANTS)
                .toItemStack();

        ItemStack pickaxe = new ItemBuilder(Material.GOLD_PICKAXE,2)
                .withEnchantment(Enchantment.DURABILITY, 10)
                .toItemStack();

        ItemStack skull = new ItemBuilder(Material.SKULL_ITEM, 1, 3).toPlayerHead("Chriis");
```

### InventoryUtils<br>
centerPositions() = Gibt eine Array mit Slots zurück, womit eine gleichmäßige Anordnung von X beliebigen Items auf eine bestimmte Inventargröße erstellt werden kann. <br>
containsItems() = Checkt in einem Player Inventar ob ein exakt gleicher ItemStack bereits vorhanden ist. <br>
getNextInventorySize() = Gibt die nächste Inventargröße (9,18,27...) für einen Integer an: 7 - 9 | 44 - 45 etc.<br>
```java
          //Beispiel centerPositions
        Inventory inventory = null;
        int inventorySize = 9*3;
        ItemStack items[] = new ItemStack[0];
        int[] slots = InventoryUtils.centerPositons(inventorySize, items.length);

        for(int i = 0; i < items.length; i++){
            ItemStack itemStack = items[i];
            int slot = slots[i];
            
            inventory.setItem(slot, itemStack);
        }
```

## Titles, Actionbars, Header und Footer
### Actionsbars
```java
        new ActionbarBuilder("§aHallo Dummy")
            .showTime(10)//Show 10 seconds
            .showAllPlayers(); //Show to all players on server

        new ActionbarBuilder("§aHallo " + player.getName())
                .showPlayer(player); //Just Show player
```
### Titles
```java
        new TitleBuilder("§aTEXT", "§bSUBTEXT")
                .fadein(1)
                .show(5)
                .fadeout(3)
                .showPlayer(player);

        new TitleBuilder("§aTEXT", "§bSUBTEXT")
                .fadein(1)
                .show(5)
                .fadeout(3)
                .showAllPlayers(); //show to all players
```
### Header und Footer
```java
        new HeaderFooterBuilder()
                .addEmptyHeaderLine()
                .addHeaderLine("§aCraftLeben.de")
                .addHeaderLine("§eBester Server")
                .addEmptyHeaderLine()
                .addEmptyFooterLine()
                .addFooterline("§cDanke fürs lesen")
                .sendPlayer(player); // oder .sendAllPlayers();
```

## CraftUser - Coins, Rang, Spielzeit usw.<br>
CraftUser können sowohl von einer UUID, einem Player oder einem ProxiedPlayer erstellt werden. Es wird wieder ein Callback benutzt, welches einen CraftUser zurück gibt, wenn der Spieler aus der Datenbank geladen wurde. Die Daten sind immer Live und müssen nie Gecached werden, da die Callbacks asynchron verlaufen. Sobald Spielerinformationen benötigt werden, muss der ganze Code der auf diese Information aufbaut, in die Callback Methode geschrieben werden. Man kann auch Callbacks in Callbacks verwenden. So kann man z.B. erst die Coins und dann die Stats abfragen. Alles asynchron. Weiteres unten bei Stats. Callbacks everywhere :D

### Alle Informationen abfragen<br>
Man verwendet die Klasse CraftUserFetcher.java um einen CraftUser über ein CraftUserCallback zu erhalten:
```java
        new CraftUserFetcher(player, craftUser -> {
            String name = craftUser.getName();
            int coins = craftUser.getCoins();
            Rank rank = craftUser.getRank();
            ...
        });
```

### Beispiel: Rang checken
```java
        new CraftUserFetcher(player, craftUser -> {
            Rank rank = craftUser.getRank();

            if(rank.getId() >= Rank.SUPPORTER.getId()){
                //Spieler ist Supporter oder höher
            }else{
                //Spieler ist nicht Supporter oder höher
            }
        });
```

### Beispiel: PlayerValues setzen und abfragen. Nützlich für Einstellungen etc.<br>
Setzen
```java 
        new CraftUserFetcher(player, user -> {
           user.setPlayerValue("lobby_autonick_enabled", true); //Funktioniert wie eine HashMap
        });
```
<br> Abfragen
```java 
     new CraftUserFetcher(player, user -> {
           user.getPlayerValue("lobby_autonick_enabled", playerValueObject -> {
               if(playerValueObject != null){
                   Boolean autonick = (Boolean) playerValueObject; //Zu dem Typ casten den man gesetzt hat (Hier: Boolean - Ist ja true / false)
                   if(autonick){
                       //Wenn AutoNick aktivert ist ....
                   }
               }else{
                   //Player Value wurde noch nie gesetzt (Also hier wäre AutoNick auch deaktiviert)
               }
           });
        });
```

### Beispiel: Auf anderen BungeeCord Server verschieben
```java 
                UUID uuid = ...;
                new CraftUserFetcher(uuid, craftUser -> {
                   craftUser.sendToServer("Lobby-1");
                });
```
## Stats<br>
Stats werden exakt gleich wie CraftUser abgefragt. Über die Klasse StatsFetcher.java kann man PlayerStats erhalten. An diesem Beispiel zeige ich gleich, was mit  "Callback in Callback" gemeint ist. Erst wird ein CraftUser angefordert, DANACH erst die Stats. Es wird ein GameType.java benötigt.

Stats abfragen:
```java
        new CraftUserFetcher(player, craftUser -> {
            new StatsFetcher(craftUser, GameType.FREEBUILD).getStats(playerStats -> {
                int kills = playerStats.getKills();
                ...
            });

        });
```

## Scoreboard<br>
Mit dem ScoreboardBuilder können einfach Scoreboards erstellt werden, die auch einfach geupdatet werden können<br>
addEmptyLine() = Fügt eine Leere Zeile hinzu<br>
addLine() = Fügt eine Zeile mit Text hinzu, die nicht geupdatet werden kann<br>
addUpdateRow() = Fügt eine Zeile Text hinzu, die geupdatet werden kann, %s wird immer durch entsprechenden Text ersetzt<br>
disabledSideboard() = Deaktiviert das Scoreboard an der Rechten Seite - Tab Prefixes bleiben erhalten!<br>
resetScores() = Setzt alle Reihen im Sideboard zurück!

Beispiel: In diesem Beispiel wird bei jeder Bewegung die aktuelle Location im Scoreboard angezeigt
```java
        Player player = ...;
                new ScoreboardBuilder(player, "§f§LCRAFTLEBEN.DE")
                        .addEmptyLine()
                        .addLine("§7Koordinaten§8:")
                        .addUpdateRow("scoreboard_x","§8» ", "§c%s")//%s Wird beim updaten ersetzt - Dazu unten mehr
                        .addUpdateRow("scoreboard_y","§8» ", "§c%s")//%s Wird beim updaten ersetzt - Dazu unten mehr
                        .addUpdateRow("scoreboard_z","§8» ", "§c%s")//%s Wird beim updaten ersetzt - Dazu unten mehr
                        .addEmptyLine()
                        .sendPlayer();


        //Score updaten
            Integer x = Double.valueOf(player.getLocation().getX()).intValue();
            Integer y = Double.valueOf(player.getLocation().getY()).intValue();
            Integer z = Double.valueOf(player.getLocation().getZ()).intValue();
            

            ScoreboardBuilder.getBuilder(player).updateRow("scoreboard_x", "", x.toString());
            ScoreboardBuilder.getBuilder(player).updateRow("scoreboard_y", "", y.toString());
            ScoreboardBuilder.getBuilder(player).updateRow("scoreboard_z", "", z.toString());
```

## ServerValues - Speichere z.B. Locations / Globale Server Einstellungen (Motd...)
Servervalues sind globale Werte, die wirklich nur auf den gesamten Server zutreffen, und nicht auf einzelne Spieler. Wenn du Spieler Einstellungen speichern möchtest, schaue dir PlayerValues an.
Ähnlich wie bei den PlayerValues ist eine ServerValue einfach gesagt ein synchronisierter HashMap Eintrag - Sprich Key -> Value<br>
In diesem Beispiel wird eine Location mithilfe von LocationParser.java (Dazu unten mehr) als ServerValue gespeichert. 
### Erstellen - Vorher checken ob dieser existiert! - Beispiel: Globale Location speichern
```java
                Location location = ...;
                ServerValue serverValue = new ServerValue("superjump_spawn_location", LocationParser.toString(location));
                serverValue.create();

```
### Auslesen / Checken ob existent
```java
                ServerValueFetcher.getServerValue("superjump_spawn_location", serverValue -> {
                    if(serverValue == null){
                        //ServerValue existiert nicht
                    }else{
                        //Servervalue existiert - Auslesen :P
                        player.sendMessage("Server Value: " + serverValue.getValue());

                    }
                });

```
### Updaten
```java
                 serverValue.update(LocationParser.toString(newLocation));

```
### Löschen
```java
                 serverValue.delete();

```

## LocationParser - Location zu einem String oder String zu Location
Diese einfache Methode ist dafür nützlich, um wie z.B. eine Location zu einem Text umzuwandeln, damit dieser dann in MySQL oder einer Config gespeichert werden kann.
### Location zum String
```java
                Location location = ...;
                String stringLocation = LocationParser.toString(location);

```
### String zu Location
```java
                String stringLocation = "...";
                Location location = LocationParser.fromString(stringLocation);

```

## Teleport
Um einen Spieler zu teleportieren, kann man den TeleportBuilder nutzen, der einfache funktionen direkt abnimmt. Teleport Delay, nicht bewegen usw. Man benötigt nur einen Locationname (z.B. "Spawn") und eine Location<br>
cancelOnMovement() = Bricht den Teleport ab, wenn man sich bewegt<br>
withDelay() = Fügt für Spieler eine 5, für Premium 3 Sekunden Wartezeit ein<br>
withMessage() = Überschreibt die Standardnachricht beim Teleport<br>
teleport() = Startet den Teleportvorgang für einen Spieler<br>
cancelTeleport() = Bricht den Teleportvorgang ab, wenn es nötig sein sollte
### Beispiel - Spawn Command, mit Delay, eigener Nachricht und Bewegung deaktiviert

```java
                Location spawn = null;
                new TeleportBuilder("Spawn", spawn)
                        .withDelay()
                        .cancelOnMovement(true)
                        .withMessage("§a§lDU BIST JETZT AM SPAWN YEAH")
                        .teleport(player);

```

## Netzwerkkommunikation
Durch die C.O.R.E Netzwerkkommunikation ist es möglich, Nachrichten von Spigot zu BungeeCord und zurück zu senden. So sind beispielsweise sofortige Rangwechsel im Scoreboard möglich, aber auch das Abfragen der Spielerzahl von einem anderen Unterserver. <br>
### Nachricht von BungeeCord an Spigot senden
```java
//Dokumantation kommt noch
```
### Nachricht von Spigot an BungeeCord senden
```java
//Dokumantation kommt noch

```
### Spielerzahl von anderem Unterserver auslesen
```java
            int players = NetworkManager.getPlayers("Lobby-1");

```

## Events
PlayerRankChangeEvent.java -> Wird getriggered wenn der Rang eines Spielers verändert wird
PlayerCoinChangeEvent.java -> Wird getriggered wenn ein Spielers Coins verdient oder verliert
CoreBungeeMessageReceiveEvent.java -> Wird getriggered, wenn ein Spigot Server eine Nachricht von BungeeCordt (Siehe Netzwerkkommunikation)
CoreMessageReceiveEvent.java (BungeeCord) -> Wird getriggered, wenn der BungeeCord eine Nachricht eines Spigot Servers erhält (Siehe Netzwerkkommunikation)

