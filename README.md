# HyternalCore Developer API Documentation

**Complete guide for building Hytale plugins with HyternalCore integration**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Java API Reference](#java-api-reference)
4. [HTTP API Reference](#http-api-reference)
5. [Complete Examples](#complete-examples)
6. [Best Practices](#best-practices)
7. [API Quick Reference](#api-quick-reference)

---

## Introduction

HyternalCore is a comprehensive MMO framework for Hytale that provides:

- **Class System**: Multi-class system with leveling, XP, and talent trees
- **Economy**: Currency management with MySQL persistence
- **Quest System**: Comprehensive quest framework with objectives and rewards
- **Party System**: Cross-server party management via Redis
- **Stats System**: MMO-style stats (Strength, Agility, Intelligence, Constitution, Wisdom, Spirit)
- **Pet System**: Pet ownership, summoning, and customization
- **Cluster Management**: Multi-server coordination and player transfers
- **HUD System**: Custom MMO HUD for players

---

## Getting Started

### Prerequisites

- Java 25 or higher
- Hytale server with HyternalCore installed
- Basic understanding of Hytale plugin development
- Access to server MySQL and Redis (for cluster features)

### Project Setup

#### 1. Create Project Structure

```
YourPlugin/
├── src/main/java/
│   └── com/yourname/yourplugin/
│       ├── YourPlugin.java
│       └── (your classes)
├── src/main/resources/
│   └── manifest.json
└── pom.xml (or build.gradle)
```

#### 2. Add Dependencies

**Maven (pom.xml):**
```xml
<dependencies>
    <!-- Hytale API -->
    <dependency>
        <groupId>com.hypixel.hytale</groupId>
        <artifactId>hytale-server-api</artifactId>
        <version>1.0.0</version>
        <scope>provided</scope>
    </dependency>
    
    <!-- HyternalCore -->
    <dependency>
        <groupId>com.mmoserver</groupId>
        <artifactId>HyternalCore</artifactId>
        <version>1.0.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

**Gradle (build.gradle):**
```gradle
dependencies {
    compileOnly 'com.hypixel.hytale:hytale-server-api:1.0.0'
    compileOnly 'com.mmoserver:HyternalCore:1.0.0'
}
```

#### 3. Configure Manifest

**manifest.json:**
```json
{
  "Group": "YourCompany",
  "Name": "YourPlugin",
  "Version": "1.0.0",
  "Description": "Your plugin description",
  "Authors": [
    {
      "Name": "Your Name"
    }
  ],
  "Website": "",
  "ServerVersion": "*",
  "Dependencies": {
    "HyternalCore": "1.0.0"
  },
  "OptionalDependencies": {},
  "DisabledByDefault": false,
  "Main": "com.yourname.yourplugin.YourPlugin",
  "IncludesAssetPack": false
}
```

**Critical:** The `"Dependencies"` field ensures HyternalCore loads before your plugin.

#### 4. Basic Plugin Class

**YourPlugin.java:**
```java
package com.yourname.yourplugin;

import com.hypixel.hytale.server.core.HytaleServer;
import com.hypixel.hytale.server.core.plugin.HytalePlugin;
import com.mmoserver.hyternalcore.api.HyternalApi;
import com.mmoserver.hyternalcore.api.HyternalApiProvider;

public class YourPlugin extends HytalePlugin {
    
    private HyternalApi api;
    
    @Override
    public void onEnable() {
        // Get HyternalCore API
        this.api = HyternalApiProvider.get();
        
        if (api == null) {
            getLogger().severe("HyternalCore API not available! Disabling plugin.");
            getServer().getPluginManager().disablePlugin(this);
            return;
        }
        
        getLogger().info("Successfully connected to HyternalCore!");
        
        // Your initialization code here
        registerListeners();
        registerCommands();
    }
    
    @Override
    public void onDisable() {
        getLogger().info("Plugin disabled.");
    }
    
    public HyternalApi getApi() {
        return api;
    }
    
    private void registerListeners() {
        // Register your event listeners
    }
    
    private void registerCommands() {
        // Register your commands
    }
}
```

---

## Java API Reference

### Accessing the API

```java
HyternalApi api = HyternalApiProvider.get();
if (api != null) {
    // API is available
}
```

### Available Services

The `HyternalApi` interface provides access to all core services:

```java
public interface HyternalApi {
    ClusterConfig getClusterConfig();
    ClusterRegistry getClusterRegistry();
    OnlinePlayerRegistry getOnlinePlayers();
    PartyService getPartyService();
    PartyMapService getPartyMapService();
    PlayerClassTracker getClassTracker();
    MMOStatsManager getStatsManager();
    CurrencyService getCurrencyService();
    QuestService getQuestService();
    QuestDialogService getQuestDialogService();
    NpcSelectionService getNpcSelectionService();
    RedisCoordinator getRedisCoordinator();
    TransferService getTransferService();
    PlayerStateService getPlayerStateService();
    HudUpdateService getHudUpdateService();
    PassiveEngine getPassiveEngine();
    MySqlCoordinator getMySqlCoordinator();
    DiscordStatusService getDiscordStatusService();
    PetManager getPetManager();
}
```

---

### 1. PlayerClassTracker - Class & Leveling System

Manages player classes, levels, experience, and talents.

#### Get Player Class Data

```java
import com.mmoserver.hyternalcore.util.PlayerClassTracker;
import com.mmoserver.hyternalcore.util.PlayerClassTracker.PlayerClassData;

UUID playerUuid = player.getUniqueId();
PlayerClassData classData = api.getClassTracker().getPlayerData(playerUuid);

if (classData != null) {
    String mainClass = classData.mainClass;        // e.g., "Fighter", "Mage"
    String subclass = classData.subclass;          // e.g., "Berserker", "Pyromancer"
    int level = classData.level;                   // Current level (1-30)
    long experience = classData.experience;        // Current XP
    Map<Integer, String> talents = classData.selectedTalents;  // Talent choices
}
```

#### Set Player Class

```java
api.getClassTracker().setPlayerClass(playerUuid, "Fighter", "Berserker");
```

#### Add Experience

```java
// Add 500 XP - automatically handles level ups
api.getClassTracker().addExperience(playerUuid, 500);

// Refresh stats after level up
api.getStatsManager().refreshStats(playerUuid);
```

#### Check if Player Has Class

```java
boolean hasClass = api.getClassTracker().hasClass(playerUuid);
```

#### Talent Management

```java
// Check if player has specific talent
boolean hasTalent = api.getClassTracker().hasTalent(playerUuid, "talent_heavy_armor");

// Set talent choice at specific level
api.getClassTracker().setTalent(playerUuid, 5, "talent_shield_bash");
```

#### Experience Calculations

```java
// Get XP needed for next level
long xpNeeded = PlayerClassTracker.getExpForNextLevel(currentLevel);

// Get total XP needed to reach a level
long totalXp = PlayerClassTracker.getTotalExpForLevel(10);  // Total XP from 1 to 10

// Get progress percentage
float progress = classData.getExpPercentage();  // Returns 0.0 to 1.0

// Get formatted display
String levelDisplay = classData.getLevelDisplay();  // "Lv. 15"
String xpDisplay = classData.getExpDisplay();      // "1250 / 2000 XP"
```

---

### 2. CurrencyService - Economy System

Manages player currency (gold) with MySQL persistence.

#### Get Balance

```java
import com.mmoserver.hyternalcore.economy.CurrencyService;

UUID playerUuid = player.getUniqueId();
String playerName = player.getDisplayName();

long balance = api.getCurrencyService().getBalance(playerUuid, playerName);
```

#### Add Currency

```java
// Add 500 gold
api.getCurrencyService().addBalance(playerUuid, playerName, 500);
```

#### Spend Currency

```java
// Try to spend 100 gold - returns true if successful
boolean success = api.getCurrencyService().trySpend(playerUuid, playerName, 100);

if (success) {
    player.sendMessage("§aPurchase successful!");
} else {
    player.sendMessage("§cNot enough gold!");
}
```

#### Example: Shop System

```java
public boolean purchaseItem(Player player, long cost) {
    UUID uuid = player.getUniqueId();
    String name = player.getDisplayName();
    
    if (api.getCurrencyService().trySpend(uuid, name, cost)) {
        player.sendMessage("§aPurchased for " + cost + " gold!");
        return true;
    } else {
        long balance = api.getCurrencyService().getBalance(uuid, name);
        player.sendMessage("§cNeed " + cost + " gold. You have: " + balance);
        return false;
    }
}
```

---

### 3. QuestService - Quest System

Comprehensive quest management with objectives, rewards, and tracking.

#### Get Player Quests

```java
import com.mmoserver.hyternalcore.quest.QuestService;
import com.mmoserver.hyternalcore.quest.QuestProgress;
import com.mmoserver.hyternalcore.quest.QuestStatus;

List<QuestProgress> quests = api.getQuestService().getProgress(playerUuid);

for (QuestProgress quest : quests) {
    String questId = quest.questId;
    QuestStatus status = quest.status;  // ACTIVE, READY, COMPLETED
    Map<String, Integer> progress = quest.progress;  // Objective progress
    long updatedAt = quest.updatedAt;
}
```

#### Get Specific Quest Progress

```java
Optional<QuestProgress> questOpt = api.getQuestService().getProgress(playerUuid, "quest_defeat_bandits");

if (questOpt.isPresent()) {
    QuestProgress quest = questOpt.get();
    // Check progress
}
```

#### Start Quest

```java
import com.hypixel.hytale.server.core.entity.entities.Player;

QuestProgress progress = api.getQuestService().startQuest(player, "quest_defeat_bandits");

if (progress != null) {
    player.sendMessage("§aQuest started!");
} else {
    player.sendMessage("§cQuest already completed or invalid!");
}
```

#### Complete Quest

```java
boolean completed = api.getQuestService().completeQuest(player, "quest_defeat_bandits");

if (completed) {
    player.sendMessage("§6Quest completed! Rewards granted.");
}
```

#### Abandon Quest

```java
boolean abandoned = api.getQuestService().abandonQuest(player, "quest_defeat_bandits");
```

#### Check Quest Status

```java
public boolean hasCompletedQuest(UUID playerUuid, String questId) {
    Optional<QuestProgress> questOpt = api.getQuestService().getProgress(playerUuid, questId);
    return questOpt.isPresent() && questOpt.get().status == QuestStatus.COMPLETED;
}
```

#### Get Quest Definition

```java
import com.mmoserver.hyternalcore.quest.QuestDefinition;

Optional<QuestDefinition> defOpt = api.getQuestService().getDefinition("quest_defeat_bandits");

if (defOpt.isPresent()) {
    QuestDefinition def = defOpt.get();
    String name = def.name;
    String description = def.description;
    List<QuestObjective> objectives = def.objectives;
    QuestReward rewards = def.rewards;
}
```

#### Track Quest Objectives

```java
// Quest system automatically tracks:
// - Kill objectives (onMobKilled)
// - Talk objectives (talkToNpc)
// - Collect objectives (refreshCollectProgress)

// Manually update quest progress if needed
api.getQuestService().refreshCollectProgress(player, "quest_gather_herbs");
```

---

### 4. PartyService - Party Management

Cross-server party system using Redis for coordination.

#### Get Player's Party

```java
import com.mmoserver.hyternalcore.cluster.PartyService;
import com.mmoserver.hyternalcore.cluster.PartyService.Party;

Optional<Party> partyOpt = api.getPartyService().getPartyByPlayer(playerUuid);

if (partyOpt.isPresent()) {
    Party party = partyOpt.get();
    String partyId = party.getId();
    UUID leaderId = party.getLeader();
    Set<UUID> members = party.getMembers();
    int size = members.size();
}
```

#### Create Party

```java
// Creates party with player as leader
Party party = api.getPartyService().createParty(playerUuid);
```

#### Invite to Party

```java
UUID leaderId = player.getUniqueId();
UUID targetId = targetPlayer.getUniqueId();

boolean invited = api.getPartyService().invite(leaderId, targetId);

if (invited) {
    player.sendMessage("§aInvite sent!");
} else {
    player.sendMessage("§cFailed to send invite. Party full or you're not leader.");
}
```

#### Accept Party Invite

```java
// Get pending invites
Set<String> invites = api.getPartyService().getInvites(playerUuid);

// Accept specific invite
boolean accepted = api.getPartyService().acceptInvite(playerUuid, partyId);
```

#### Leave Party

```java
api.getPartyService().leaveParty(playerUuid);
```

#### Disband Party

```java
// Only leader can disband
api.getPartyService().disbandParty(leaderUuid);
```

#### Check if Players in Same Party

```java
public boolean areInSameParty(UUID player1, UUID player2) {
    Optional<Party> party1 = api.getPartyService().getPartyByPlayer(player1);
    Optional<Party> party2 = api.getPartyService().getPartyByPlayer(player2);
    
    if (party1.isPresent() && party2.isPresent()) {
        return party1.get().getId().equals(party2.get().getId());
    }
    return false;
}
```

#### Get Party Count

```java
int totalParties = api.getPartyService().getPartyCount();
```

---

### 5. MMOStatsManager - Stats System

Manages player stats including base, equipment, and buff modifiers.

#### Get Player Stats

```java
import com.mmoserver.hyternalcore.stats.MMOStatsManager;
import com.mmoserver.hyternalcore.stats.PlayerStats;

PlayerStats stats = api.getStatsManager().getStats(playerUuid);

// Get individual stat
int strength = api.getStatsManager().getStat(playerUuid, "strength");
int agility = api.getStatsManager().getStat(playerUuid, "agility");
int intelligence = api.getStatsManager().getStat(playerUuid, "intelligence");
int constitution = api.getStatsManager().getStat(playerUuid, "constitution");
int wisdom = api.getStatsManager().getStat(playerUuid, "wisdom");
int spirit = api.getStatsManager().getStat(playerUuid, "spirit");

// Get all stats
Map<String, Integer> allStats = api.getStatsManager().getAllStats(playerUuid);
```

#### Get Combat Stats

```java
import com.mmoserver.hyternalcore.stats.PlayerStats.CombatStats;

CombatStats combat = api.getStatsManager().getCombatStats(playerUuid);

int maxHealth = combat.maxHealth;
int maxMana = combat.maxMana;
double healthRegen = combat.healthRegen;
double manaRegen = combat.manaRegen;
double physicalDamage = combat.physicalDamage;
double spellPower = combat.spellPower;
double critChance = combat.critChance;
double critDamage = combat.critDamage;
double dodge = combat.dodge;
double armor = combat.armor;
double magicResist = combat.magicResist;
double healingPower = combat.healingPower;
```

#### Apply Stat Buff

```java
// Add temporary buff: +10 Strength for 60 seconds
api.getStatsManager().addBuff(playerUuid, "strength", 10);

// Buff stays until manually removed
```

#### Remove Stat Buff

```java
// Remove buff
api.getStatsManager().removeBuff(playerUuid, "strength", 10);
```

#### Equipment Stats

```java
// Add equipment stat bonus
api.getStatsManager().addEquipmentStat(playerUuid, "strength", 5);

// Clear all equipment stats (when unequipping)
api.getStatsManager().clearEquipmentStats(playerUuid);
```

#### Refresh Stats

```java
// Recalculate stats after level up or class change
api.getStatsManager().refreshStats(playerUuid);
```

#### Get Formatted Stats Display

```java
String statsDisplay = api.getStatsManager().getStatsDisplay(playerUuid);
player.sendMessage(statsDisplay);
```

**Stat IDs:**
- `"strength"` - Physical damage, health
- `"agility"` - Dodge, crit chance
- `"intelligence"` - Spell power, mana
- `"constitution"` - Max health, health regen
- `"wisdom"` - Max mana, mana regen
- `"spirit"` - Healing power, magic resist

---

### 6. PetManager - Pet System

Pet ownership, summoning, and customization.

#### Get Player's Pet Inventory

```java
import com.mmoserver.hyternalcore.pet.PetManager;
import com.mmoserver.hyternalcore.pet.PlayerPetInventory;
import com.mmoserver.hyternalcore.pet.PetEntry;

PetManager petManager = api.getPetManager();
PlayerPetInventory inventory = petManager.getInventory(playerUuid);

// Get all unlocked pets
List<PetEntry> pets = inventory.getPets();

for (PetEntry pet : pets) {
    String petTypeId = pet.getPetTypeId();      // e.g., "pet_wolf"
    String displayName = pet.getDisplayName();  // Custom name
    float scale = pet.getScale();               // Size (default 0.6)
    String groupType = pet.getGroupType();      // NPC group
}
```

#### Unlock Pet

```java
boolean unlocked = petManager.unlockPet(playerUuid, "pet_wolf");

if (unlocked) {
    player.sendMessage("§aPet unlocked!");
}
```

#### Get Active Pet

```java
import com.mmoserver.hyternalcore.pet.ActivePet;

ActivePet activePet = petManager.getActivePet(playerUuid);

if (activePet != null) {
    String petTypeId = activePet.getPetTypeId();
    String petName = activePet.getPetName();
}
```

#### Check if Pet is Disabled

```java
boolean disabled = petManager.isPetDisabled("pet_dragon");
```

---

### 7. TransferService - Server Transfers

Transfer players between servers in the cluster.

#### Transfer Player

```java
import com.mmoserver.hyternalcore.cluster.TransferService;
import com.mmoserver.hyternalcore.cluster.ClusterNode;
import com.hypixel.hytale.server.core.entity.entities.Player;

// Get target server
Optional<ClusterNode> targetOpt = api.getRedisCoordinator().getOnlineNode("hub-1");

if (targetOpt.isPresent()) {
    api.getTransferService().routePlayer(player, targetOpt.get(), "Going to hub");
}
```

---

### 8. OnlinePlayerRegistry - Player Management

Access online players and their data.

#### Get Online Player

```java
import com.hypixel.hytale.server.core.entity.entities.Player;

Player player = api.getOnlinePlayers().getPlayer(playerUuid);
```

#### Find Player by Name

```java
Optional<Player> playerOpt = api.getOnlinePlayers().findByName("PlayerName");
```

#### Get All Online Players

```java
Collection<Player> players = api.getOnlinePlayers().getPlayers();
```

#### Get Online Count

```java
int count = api.getOnlinePlayers().count();
```

---

### 9. ClusterConfig & Registry

Access cluster configuration and server information.

#### Get Server Info

```java
import com.mmoserver.hyternalcore.cluster.ClusterConfig;

ClusterConfig config = api.getClusterConfig();

String serverId = config.getServerId();
String group = config.getGroup();
String region = config.getRegion();
int partyMaxSize = config.getPartyMaxSize();
```

#### Get Online Servers

```java
import com.mmoserver.hyternalcore.cluster.ClusterRegistry;
import com.mmoserver.hyternalcore.cluster.ClusterNode;

ClusterRegistry registry = api.getClusterRegistry();
List<ClusterNode> nodes = registry.getOnlineNodes();

for (ClusterNode node : nodes) {
    String serverId = node.getServerId();
    String address = node.getAddress();
    int port = node.getPort();
    int currentPlayers = node.getCurrentPlayers();
    int maxPlayers = node.getMaxPlayers();
}
```

---

## HTTP API Reference

The HTTP API provides RESTful access to HyternalCore features for external tools, web dashboards, and automation.

### Configuration

Edit `HyternalCore/sharding.properties`:

```properties
api.enabled=true
api.host=127.0.0.1
api.port=8080
api.key=your-secret-key-here
```

### Authentication

Include one of these headers with every request:

```
X-API-Key: your-secret-key-here
```

or

```
Authorization: Bearer your-secret-key-here
```

### Base URL

```
http://127.0.0.1:8080/api
```

---

### Endpoints

#### Server Status

```http
GET /api/status
```

**Response:**
```json
{
  "serverId": "game-1",
  "group": "production",
  "region": "us-east",
  "uptimeMillis": 3600000,
  "onlineCount": 42,
  "partyCount": 8,
  "nodes": 3
}
```

---

#### Get All Players

```http
GET /api/players
```

**Response:**
```json
[
  {
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "online": true,
    "name": "PlayerName",
    "class": {
      "mainClass": "Fighter",
      "subclass": "Berserker",
      "level": 25,
      "experience": 12500,
      "createdAt": "2026-01-01T00:00:00Z",
      "lastPlayed": "2026-01-17T12:00:00Z",
      "talents": {"5": "talent_shield_bash", "10": "talent_heavy_armor"}
    },
    "stats": {
      "base": {
        "strength": 85,
        "agility": 42,
        "intelligence": 30,
        "constitution": 75,
        "wisdom": 35,
        "spirit": 40
      },
      "combat": {
        "maxHealth": 1250,
        "maxMana": 450,
        "healthRegen": 2.5,
        "manaRegen": 1.8,
        "physicalDamage": 125.5,
        "spellPower": 45.0,
        "critChance": 0.15,
        "critDamage": 1.5,
        "dodge": 0.10,
        "armor": 85.0,
        "magicResist": 40.0,
        "healingPower": 1.0
      }
    },
    "currency": 5000,
    "quests": [
      {
        "questId": "quest_defeat_bandits",
        "status": "ACTIVE",
        "progress": {"obj_1": 5},
        "updatedAt": 1705497600000
      }
    ],
    "party": {
      "id": "party-uuid",
      "leader": "leader-uuid",
      "members": ["member-uuid-1", "member-uuid-2"],
      "createdAt": 1705497600000,
      "updatedAt": 1705497600000
    },
    "pets": {
      "inventory": [
        {
          "petTypeId": "pet_wolf",
          "displayName": "Fluffy",
          "scale": 0.6,
          "groupType": "default"
        }
      ],
      "active": {
        "petTypeId": "pet_wolf",
        "petName": "Fluffy"
      }
    }
  }
]
```

---

#### Get Player by ID/Name

```http
GET /api/players/{idOrName}
```

**Example:**
```bash
curl -H "X-API-Key: your-key" \
  http://127.0.0.1:8080/api/players/PlayerName
```

---

#### Add/Spend Currency

```http
POST /api/players/{idOrName}/currency
Content-Type: application/json

{
  "action": "add",
  "amount": 500,
  "name": "PlayerName"
}
```

**Actions:** `add` or `spend`

**Example:**
```bash
curl -X POST \
  -H "X-API-Key: your-key" \
  -H "Content-Type: application/json" \
  -d '{"action":"add","amount":500,"name":"PlayerName"}' \
  http://127.0.0.1:8080/api/players/PlayerName/currency
```

---

#### Set Player Class

```http
POST /api/players/{idOrName}/class
Content-Type: application/json

{
  "mainClass": "Fighter",
  "subclass": "Berserker"
}
```

---

#### Add Experience

```http
POST /api/players/{idOrName}/xp
Content-Type: application/json

{
  "amount": 250
}
```

---

#### Quest Management

```http
GET /api/players/{idOrName}/quests
POST /api/players/{idOrName}/quests/start
POST /api/players/{idOrName}/quests/complete
POST /api/players/{idOrName}/quests/abandon

# Body for quest operations:
{
  "questId": "quest_defeat_bandits"
}
```

---

#### Apply/Remove Stat Buffs

```http
POST /api/players/{idOrName}/stats/buff
POST /api/players/{idOrName}/stats/buff-remove

# Body:
{
  "statId": "strength",
  "value": 10
}
```

---

#### Get Quest Definitions

```http
GET /api/quests
GET /api/quests/{questId}
```

---

#### Party Management

```http
GET /api/party/by-player/{idOrName}

POST /api/party/invite
{
  "leaderId": "uuid-or-name",
  "targetId": "uuid-or-name"
}

POST /api/party/accept
{
  "targetId": "uuid-or-name",
  "partyId": "party-uuid"
}

POST /api/party/leave
{
  "playerId": "uuid-or-name"
}

POST /api/party/disband
{
  "leaderId": "uuid-or-name"
}
```

---

#### Pet Management

```http
GET /api/pets/{idOrName}

POST /api/pets/{idOrName}/unlock
{
  "petId": "pet_wolf"
}
```

---

#### Server Transfer

```http
POST /api/transfer
{
  "playerId": "uuid-or-name",
  "serverId": "hub-1",
  "reason": "api transfer"
}
```

---

#### Cluster Information

```http
GET /api/cluster/nodes
```

**Response:**
```json
[
  {
    "serverId": "game-1",
    "group": "production",
    "region": "us-east",
    "address": "127.0.0.1",
    "port": 25565,
    "maxPlayers": 100,
    "currentPlayers": 42,
    "loadRatio": 0.42
  }
]
```

---

#### Script Execution

```http
GET /api/scripts

POST /api/scripts/run
{
  "name": "reward_players",
  "args": ["PlayerName", "500"]
}
```

---

## Complete Examples

### Example 1: Minigame Reward System

```java
package com.yourname.minigame;

import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.server.core.event.EventHandler;
import com.hypixel.hytale.server.core.event.Listener;
import com.mmoserver.hyternalcore.api.HyternalApi;
import java.util.UUID;

public class MinigameRewardListener implements Listener {
    
    private final HyternalApi api;
    
    public MinigameRewardListener(HyternalApi api) {
        this.api = api;
    }
    
    @EventHandler
    public void onGameWin(GameWinEvent event) {
        Player winner = event.getWinner();
        UUID uuid = winner.getUniqueId();
        String name = winner.getDisplayName();
        
        // Reward gold
        api.getCurrencyService().addBalance(uuid, name, 100);
        
        // Reward XP
        api.getClassTracker().addExperience(uuid, 50);
        
        // Apply temporary strength buff (5 minutes = 300 seconds)
        api.getStatsManager().addBuff(uuid, "strength", 5);
        
        // Refresh stats
        api.getStatsManager().refreshStats(uuid);
        
        winner.sendMessage("§6===================");
        winner.sendMessage("§e§lVICTORY!");
        winner.sendMessage("§a+100 Gold");
        winner.sendMessage("§a+50 XP");
        winner.sendMessage("§a+5 Strength (5 min)");
        winner.sendMessage("§6===================");
    }
}
```

---

### Example 2: Party-Based Boss Fight

```java
package com.yourname.bossfight;

import com.hypixel.hytale.server.core.entity.entities.Player;
import com.mmoserver.hyternalcore.api.HyternalApi;
import com.mmoserver.hyternalcore.cluster.PartyService.Party;
import java.util.Optional;
import java.util.UUID;

public class BossRewardHandler {
    
    private final HyternalApi api;
    
    public BossRewardHandler(HyternalApi api) {
        this.api = api;
    }
    
    public void onBossKilled(Player killer) {
        UUID killerUuid = killer.getUniqueId();
        Optional<Party> partyOpt = api.getPartyService().getPartyByPlayer(killerUuid);
        
        if (partyOpt.isPresent()) {
            // Party kill - reward all members
            Party party = partyOpt.get();
            
            for (UUID memberUuid : party.getMembers()) {
                rewardPlayer(memberUuid, 500, 250);
                
                Player member = api.getOnlinePlayers().getPlayer(memberUuid);
                if (member != null) {
                    member.sendMessage("§6Boss defeated! +500 gold, +250 XP");
                }
            }
        } else {
            // Solo kill - bonus rewards
            rewardPlayer(killerUuid, 1000, 500);
            killer.sendMessage("§6§lSOLO BOSS KILL!");
            killer.sendMessage("§a+1000 Gold (Bonus)");
            killer.sendMessage("§a+500 XP (Bonus)");
        }
    }
    
    private void rewardPlayer(UUID uuid, long gold, long xp) {
        Player player = api.getOnlinePlayers().getPlayer(uuid);
        String name = player != null ? player.getDisplayName() : "";
        
        api.getCurrencyService().addBalance(uuid, name, gold);
        api.getClassTracker().addExperience(uuid, xp);
        api.getStatsManager().refreshStats(uuid);
    }
}
```

---

### Example 3: Class-Specific Quest Requirements

```java
package com.yourname.quests;

import com.hypixel.hytale.server.core.entity.entities.Player;
import com.mmoserver.hyternalcore.api.HyternalApi;
import com.mmoserver.hyternalcore.util.PlayerClassTracker.PlayerClassData;
import java.util.UUID;

public class QuestRequirementChecker {
    
    private final HyternalApi api;
    
    public QuestRequirementChecker(HyternalApi api) {
        this.api = api;
    }
    
    public boolean canStartQuest(Player player, String questId) {
        UUID uuid = player.getUniqueId();
        PlayerClassData classData = api.getClassTracker().getPlayerData(uuid);
        
        if (classData == null) {
            player.sendMessage("§cYou must select a class first!");
            return false;
        }
        
        switch (questId) {
            case "quest_warriors_trial":
                // Requires Fighter class, level 10+
                if (!"Fighter".equals(classData.mainClass)) {
                    player.sendMessage("§cThis quest is only for Fighters!");
                    return false;
                }
                if (classData.level < 10) {
                    player.sendMessage("§cYou must be level 10 or higher!");
                    return false;
                }
                break;
                
            case "quest_arcane_secrets":
                // Requires Mage class, level 15+, specific talent
                if (!"Mage".equals(classData.mainClass)) {
                    player.sendMessage("§cThis quest is only for Mages!");
                    return false;
                }
                if (classData.level < 15) {
                    player.sendMessage("§cYou must be level 15 or higher!");
                    return false;
                }
                if (!api.getClassTracker().hasTalent(uuid, "talent_spellweaver")) {
                    player.sendMessage("§cYou need the Spellweaver talent!");
                    return false;
                }
                break;
        }
        
        return true;
    }
}
```

---

### Example 4: Shop with Currency System

```java
package com.yourname.shop;

import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.server.core.item.ItemStack;
import com.mmoserver.hyternalcore.api.HyternalApi;
import java.util.UUID;

public class ShopManager {
    
    private final HyternalApi api;
    
    public ShopManager(HyternalApi api) {
        this.api = api;
    }
    
    public boolean purchaseItem(Player player, ItemStack item, long cost) {
        UUID uuid = player.getUniqueId();
        String name = player.getDisplayName();
        
        // Check balance
        long balance = api.getCurrencyService().getBalance(uuid, name);
        
        if (balance < cost) {
            player.sendMessage("§c═══════════════════════");
            player.sendMessage("§c§lINSUFFICIENT FUNDS");
            player.sendMessage("§7Cost: §6" + cost + " gold");
            player.sendMessage("§7Your balance: §6" + balance + " gold");
            player.sendMessage("§7Need: §c" + (cost - balance) + " more gold");
            player.sendMessage("§c═══════════════════════");
            return false;
        }
        
        // Try to spend
        if (api.getCurrencyService().trySpend(uuid, name, cost)) {
            // Give item to player
            player.getInventory().addItem(item);
            
            long newBalance = api.getCurrencyService().getBalance(uuid, name);
            
            player.sendMessage("§a═══════════════════════");
            player.sendMessage("§a§lPURCHASE SUCCESSFUL");
            player.sendMessage("§7Spent: §6" + cost + " gold");
            player.sendMessage("§7New balance: §6" + newBalance + " gold");
            player.sendMessage("§a═══════════════════════");
            return true;
        }
        
        return false;
    }
    
    public void sellItem(Player player, ItemStack item, long value) {
        UUID uuid = player.getUniqueId();
        String name = player.getDisplayName();
        
        // Remove item
        player.getInventory().removeItem(item);
        
        // Add currency
        api.getCurrencyService().addBalance(uuid, name, value);
        
        long balance = api.getCurrencyService().getBalance(uuid, name);
        
        player.sendMessage("§a═══════════════════════");
        player.sendMessage("§a§lITEM SOLD");
        player.sendMessage("§7Received: §6" + value + " gold");
        player.sendMessage("§7New balance: §6" + balance + " gold");
        player.sendMessage("§a═══════════════════════");
    }
}
```

---

### Example 5: Achievement System

```java
package com.yourname.achievements;

import com.hypixel.hytale.server.core.entity.entities.Player;
import com.mmoserver.hyternalcore.api.HyternalApi;
import java.util.UUID;

public class AchievementManager {
    
    private final HyternalApi api;
    
    public AchievementManager(HyternalApi api) {
        this.api = api;
    }
    
    public void grantAchievement(Player player, String achievementId) {
        UUID uuid = player.getUniqueId();
        String name = player.getDisplayName();
        
        switch (achievementId) {
            case "first_kill":
                api.getCurrencyService().addBalance(uuid, name, 50);
                api.getClassTracker().addExperience(uuid, 25);
                
                player.sendMessage("§6★★★★★★★★★★★★★★★★★★★★★★");
                player.sendMessage("§e§lACHIEVEMENT UNLOCKED!");
                player.sendMessage("§f§lFirst Blood");
                player.sendMessage("§7Defeat your first enemy");
                player.sendMessage("§a+50 Gold | +25 XP");
                player.sendMessage("§6★★★★★★★★★★★★★★★★★★★★★★");
                break;
                
            case "level_10":
                api.getCurrencyService().addBalance(uuid, name, 500);
                api.getPetManager().unlockPet(uuid, "pet_wolf");
                
                player.sendMessage("§6★★★★★★★★★★★★★★★★★★★★★★");
                player.sendMessage("§e§lACHIEVEMENT UNLOCKED!");
                player.sendMessage("§f§lReaching New Heights");
                player.sendMessage("§7Reach level 10");
                player.sendMessage("§a+500 Gold | Pet Unlocked: §fWolf");
                player.sendMessage("§6★★★★★★★★★★★★★★★★★★★★★★");
                break;
                
            case "party_leader":
                // 30 minutes = 1800 seconds
                api.getStatsManager().addBuff(uuid, "wisdom", 10);
                
                player.sendMessage("§6★★★★★★★★★★★★★★★★★★★★★★");
                player.sendMessage("§e§lACHIEVEMENT UNLOCKED!");
                player.sendMessage("§f§lNatural Leader");
                player.sendMessage("§7Lead a successful party");
                player.sendMessage("§a+10 Wisdom (30 minutes)");
                player.sendMessage("§6★★★★★★★★★★★★★★★★★★★★★★");
                break;
                
            case "quest_master":
                api.getCurrencyService().addBalance(uuid, name, 1000);
                api.getClassTracker().addExperience(uuid, 500);
                api.getStatsManager().addBuff(uuid, "strength", 5);
                api.getStatsManager().addBuff(uuid, "intelligence", 5);
                
                player.sendMessage("§6★★★★★★★★★★★★★★★★★★★★★★");
                player.sendMessage("§e§lACHIEVEMENT UNLOCKED!");
                player.sendMessage("§f§lQuest Master");
                player.sendMessage("§7Complete 50 quests");
                player.sendMessage("§a+1000 Gold | +500 XP");
                player.sendMessage("§a+5 STR & INT (Permanent)");
                player.sendMessage("§6★★★★★★★★★★★★★★★★★★★★★★");
                break;
        }
        
        api.getStatsManager().refreshStats(uuid);
    }
}
```

---

### Example 6: Arena PvP System

```java
package com.yourname.arena;

import com.hypixel.hytale.server.core.entity.entities.Player;
import com.hypixel.hytale.server.core.event.EventHandler;
import com.hypixel.hytale.server.core.event.Listener;
import com.hypixel.hytale.server.core.event.player.PlayerDeathEvent;
import com.mmoserver.hyternalcore.api.HyternalApi;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

public class ArenaSystem implements Listener {
    
    private final HyternalApi api;
    private final Map<UUID, Integer> killStreaks = new HashMap<>();
    private final Map<UUID, Integer> arenaKills = new HashMap<>();
    
    public ArenaSystem(HyternalApi api) {
        this.api = api;
    }
    
    @EventHandler
    public void onPlayerDeath(PlayerDeathEvent event) {
        Player victim = event.getPlayer();
        Player killer = event.getKiller();
        
        if (killer == null || !isInArena(killer) || !isInArena(victim)) {
            return;
        }
        
        UUID killerUuid = killer.getUniqueId();
        UUID victimUuid = victim.getUniqueId();
        String killerName = killer.getDisplayName();
        
        // Reset victim's streak
        killStreaks.remove(victimUuid);
        
        // Increment killer's streak
        int streak = killStreaks.getOrDefault(killerUuid, 0) + 1;
        killStreaks.put(killerUuid, streak);
        
        // Track total kills
        int totalKills = arenaKills.getOrDefault(killerUuid, 0) + 1;
        arenaKills.put(killerUuid, totalKills);
        
        // Base rewards
        long goldReward = 25;
        long xpReward = 10;
        
        // Streak bonuses
        if (streak == 5) {
            goldReward += 50;
            broadcastArena("§6" + killerName + " §eis on a §6Killing Spree§e! (5 kills)");
        } else if (streak == 10) {
            goldReward += 100;
            api.getStatsManager().addBuff(killerUuid, "agility", 5);
            broadcastArena("§6" + killerName + " §eis §c§lDOMINATING§e! (10 kills)");
        } else if (streak >= 15) {
            goldReward += 200;
            api.getStatsManager().addBuff(killerUuid, "strength", 10);
            broadcastArena("§6" + killerName + " §eis §4§lUNSTOPPABLE§e! (15+ kills)");
        }
        
        // Award rewards
        api.getCurrencyService().addBalance(killerUuid, killerName, goldReward);
        api.getClassTracker().addExperience(killerUuid, xpReward);
        api.getStatsManager().refreshStats(killerUuid);
        
        // Notify killer
        killer.sendMessage("§e+§6" + goldReward + " gold §e| +§a" + xpReward + " XP");
        killer.sendMessage("§7Kill Streak: §6" + streak + " §7| Total Kills: §6" + totalKills);
    }
    
    private boolean isInArena(Player player) {
        // Check if player is in arena world
        return player.getWorld().getName().equals("arena");
    }
    
    private void broadcastArena(String message) {
        for (Player player : api.getOnlinePlayers().getPlayers()) {
            if (isInArena(player)) {
                player.sendMessage(message);
            }
        }
    }
    
    public void resetStats(UUID playerUuid) {
        killStreaks.remove(playerUuid);
        arenaKills.remove(playerUuid);
    }
}
```

---

## Best Practices

### 1. Always Check for Null/Empty

```java
// Good ✓
PlayerClassData classData = api.getClassTracker().getPlayerData(uuid);
if (classData != null) {
    String mainClass = classData.mainClass;
    // Use data
}

// Bad ✗ - Can throw NullPointerException
String mainClass = api.getClassTracker().getPlayerData(uuid).mainClass;
```

### 2. Use Optional Properly

```java
// Good ✓
Optional<Party> partyOpt = api.getPartyService().getPartyByPlayer(uuid);
if (partyOpt.isPresent()) {
    Party party = partyOpt.get();
    // Use party
}

// Bad ✗ - Can throw NoSuchElementException
Party party = api.getPartyService().getPartyByPlayer(uuid).get();
```

### 3. Cache API Instance

```java
public class YourPlugin extends HytalePlugin {
    private HyternalApi api;
    
    @Override
    public void onEnable() {
        this.api = HyternalApiProvider.get();
        // Store and reuse
    }
    
    public HyternalApi getApi() {
        return api;
    }
}
```

### 4. Handle API Unavailability

```java
@Override
public void onEnable() {
    this.api = HyternalApiProvider.get();
    
    if (api == null) {
        getLogger().severe("═══════════════════════════════");
        getLogger().severe("HyternalCore not found!");
        getLogger().severe("Please install HyternalCore");
        getLogger().severe("═══════════════════════════════");
        getServer().getPluginManager().disablePlugin(this);
        return;
    }
    
    getLogger().info("Connected to HyternalCore successfully");
}
```

### 5. Refresh Stats After Changes

```java
// Always refresh stats after:
// - Level ups
// - Class changes
// - Stat modifications

api.getClassTracker().addExperience(uuid, 500);
api.getStatsManager().refreshStats(uuid);  // Important!

api.getStatsManager().addBuff(uuid, "strength", 10);
api.getStatsManager().refreshStats(uuid);  // Important!
```

### 6. Validate Input

```java
public void rewardPlayer(UUID uuid, long amount) {
    if (uuid == null) {
        throw new IllegalArgumentException("UUID cannot be null");
    }
    if (amount <= 0) {
        throw new IllegalArgumentException("Amount must be positive");
    }
    
    Player player = api.getOnlinePlayers().getPlayer(uuid);
    if (player == null) {
        return;  // Player offline
    }
    
    api.getCurrencyService().addBalance(uuid, player.getDisplayName(), amount);
}
```

### 7. Use Player Name for Currency Operations

```java
// Good ✓ - Includes player name
String name = player.getDisplayName();
api.getCurrencyService().addBalance(uuid, name, 100);

// Acceptable but not ideal - Empty name
api.getCurrencyService().addBalance(uuid, "", 100);
```

### 8. Log Important Actions

```java
public void transferPlayer(Player player, String targetServer) {
    UUID uuid = player.getUniqueId();
    
    getLogger().info("Transferring player " + player.getDisplayName() + 
                    " (" + uuid + ") to " + targetServer);
    
    Optional<ClusterNode> targetOpt = api.getRedisCoordinator().getOnlineNode(targetServer);
    if (targetOpt.isPresent()) {
        api.getTransferService().routePlayer(player, targetOpt.get(), "Plugin transfer");
    } else {
        getLogger().warning("Target server " + targetServer + " not found!");
    }
}
```

### 9. Handle Quest Operations Safely

```java
// Quest operations require online player
public boolean startQuest(UUID uuid, String questId) {
    Player player = api.getOnlinePlayers().getPlayer(uuid);
    if (player == null) {
        getLogger().warning("Cannot start quest - player offline");
        return false;
    }
    
    QuestProgress progress = api.getQuestService().startQuest(player, questId);
    return progress != null;
}
```

### 10. Proper Dependency Declaration

Always declare HyternalCore in your manifest.json:

```json
{
  "Dependencies": {
    "HyternalCore": "1.0.0"
  }
}
```

This ensures:
- HyternalCore loads first
- Your plugin won't load if HyternalCore is missing
- Proper initialization order

---

## API Quick Reference

### PlayerClassTracker

```java
// Get class data
PlayerClassData data = api.getClassTracker().getPlayerData(uuid);

// Check if has class
boolean hasClass = api.getClassTracker().hasClass(uuid);

// Set class
api.getClassTracker().setPlayerClass(uuid, "Fighter", "Berserker");

// Add XP
api.getClassTracker().addExperience(uuid, 500);

// Talent management
boolean hasTalent = api.getClassTracker().hasTalent(uuid, "talent_id");
api.getClassTracker().setTalent(uuid, 5, "talent_id");

// XP calculations
long xpNeeded = PlayerClassTracker.getExpForNextLevel(level);
long totalXp = PlayerClassTracker.getTotalExpForLevel(10);
```

### CurrencyService

```java
// Get balance
long balance = api.getCurrencyService().getBalance(uuid, name);

// Add currency
api.getCurrencyService().addBalance(uuid, name, amount);

// Spend currency
boolean success = api.getCurrencyService().trySpend(uuid, name, amount);
```

### QuestService

```java
// Get quests
List<QuestProgress> quests = api.getQuestService().getProgress(uuid);
Optional<QuestProgress> quest = api.getQuestService().getProgress(uuid, questId);

// Manage quests
QuestProgress progress = api.getQuestService().startQuest(player, questId);
boolean completed = api.getQuestService().completeQuest(player, questId);
boolean abandoned = api.getQuestService().abandonQuest(player, questId);

// Get definitions
Optional<QuestDefinition> def = api.getQuestService().getDefinition(questId);
List<QuestDefinition> defs = api.getQuestService().getDefinitions();
```

### PartyService

```java
// Get party
Optional<Party> party = api.getPartyService().getPartyByPlayer(uuid);

// Create party
Party party = api.getPartyService().createParty(leaderUuid);

// Manage party
boolean invited = api.getPartyService().invite(leaderUuid, targetUuid);
boolean accepted = api.getPartyService().acceptInvite(targetUuid, partyId);
api.getPartyService().leaveParty(uuid);
api.getPartyService().disbandParty(leaderUuid);

// Get invites
Set<String> invites = api.getPartyService().getInvites(uuid);
```

### MMOStatsManager

```java
// Get stats
PlayerStats stats = api.getStatsManager().getStats(uuid);
int strength = api.getStatsManager().getStat(uuid, "strength");
Map<String, Integer> allStats = api.getStatsManager().getAllStats(uuid);
CombatStats combat = api.getStatsManager().getCombatStats(uuid);

// Buffs
api.getStatsManager().addBuff(uuid, "strength", 10);
api.getStatsManager().removeBuff(uuid, "strength", 10);

// Equipment
api.getStatsManager().addEquipmentStat(uuid, "strength", 5);
api.getStatsManager().clearEquipmentStats(uuid);

// Refresh
api.getStatsManager().refreshStats(uuid);
```

### PetManager

```java
// Get inventory
PlayerPetInventory inventory = api.getPetManager().getInventory(uuid);
List<PetEntry> pets = inventory.getPets();

// Unlock pet
boolean unlocked = api.getPetManager().unlockPet(uuid, "pet_wolf");

// Active pet
ActivePet active = api.getPetManager().getActivePet(uuid);
```

### OnlinePlayerRegistry

```java
// Get players
Player player = api.getOnlinePlayers().getPlayer(uuid);
Optional<Player> playerOpt = api.getOnlinePlayers().findByName("PlayerName");
Collection<Player> players = api.getOnlinePlayers().getPlayers();
int count = api.getOnlinePlayers().count();
```

### ClusterConfig & Registry

```java
// Config
ClusterConfig config = api.getClusterConfig();
String serverId = config.getServerId();
String group = config.getGroup();

// Registry
List<ClusterNode> nodes = api.getClusterRegistry().getOnlineNodes();
```

---

## Troubleshooting

### Common Issues

**API returns null:**
- Ensure HyternalCore is installed
- Check manifest.json has proper dependency
- Verify HyternalCore loaded before your plugin

**Player data not found:**
- Player may not have logged in yet
- Always check for null/empty
- Some operations require online players

**Quest operations fail:**
- Quest start/complete/abandon require online player
- Check if quest definition exists

**Stats not updating:**
- Call `refreshStats()` after XP/level/class changes
- Stats are calculated based on current class/level

**HTTP API permission denied:**
- Check `api.key` in `sharding.properties`
- Verify correct header (`X-API-Key` or `Authorization`)
- Ensure `api.enabled=true`

---

**Version:** 1.0.0  
**Last Updated:** January 2026  
**Platform:** Hytale Server  
**Target:** Trial Developers & Plugin Developers
