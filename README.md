Index: dist/game/config/Services.properties
===================================================================
--- dist/game/config/Services.properties	(revision 0)
+++ dist/game/config/Services.properties	(working copy)
@@ -0,0 +1,55 @@
+# ---------------------------------------------------------------------------
+# Common Service Options
+# ---------------------------------------------------------------------------
+AllowServiceVoiced = False
+
+# ---------------------------------------------------------------------------
+# Premium System Options
+# ---------------------------------------------------------------------------
+# Allow premium service system. Check other options and rates carefully, before enable it!
+# Default: False
+PremiumServiceEnabled = False
+
+# Calculate drop and spoil in party by premium rules, if last attacker is not premium owner, but there is premium owner in party.
+# Default: False
+PremiumPartyDropSpoil = False
+
+# Show premium status (a square around player's level).
+# Default: True
+ShowPremiumStatus = True
+
+# Notify player about soon premium service expiration (one day before) on enter world.
+# Default: True
+NotifyPremiumExpiration = True
+
+# Activate premium service for newbies for given number of days. Put 0 to disable feature.
+# Default: 0.
+NewbiesPremiumPeriod = 0
+
+# Allow adding premium service using .addpremium voiced command
+PremiumAllowVoiced = False
+
+# ---------------------------------------------------------------------------
+# Premium Service Price
+# ---------------------------------------------------------------------------
+# Price of premium service. Format is timePeriod,item,itemCount[;timePeriod,item,itemCount]
+# Time period - a number and code letter (s - seconds, m - minutes, h - hours, d - days, M - months).
+# If itemCount is 0 - service is free. You can define both base period (i.e. 1 hour, 1 day) and multiplied period (3 hours, 5 days)
+AccountPremiumPrice = 1s,57,0;1m,57,0;1h,57,1;1d,57,24;1M,57,720
+CharPremiumPrice = 1s,57,0;1m,57,0;1h,57,1;1d,57,24;1M,57,720
+
+# ---------------------------------------------------------------------------
+# Premium Base Rates
+# ---------------------------------------------------------------------------
+# Experience multiplier
+PremiumRateXp = 2.
+# Skill points multiplier
+PremiumRateSp = 2.
+# Item's drop multiplier
+PremiumRateDropItems = 2.
+# Item's drop multiplier for Raid Bosses
+PremiumRateRaidDropItems = 1.
+# Item's spoil multiplier
+PremiumRateSpoil = 2.
+# List of items affected by custom drop rate by id, used now for Adena rate too.
+PremiumRateDropItemsById = 57,2
\ No newline at end of file
Index: java/com/l2jserver/Config.java
===================================================================
--- java/com/l2jserver/Config.java	(revision 6303)
+++ java/com/l2jserver/Config.java	(working copy)
@@ -52,6 +52,7 @@
 import org.w3c.dom.NamedNodeMap;
 import org.w3c.dom.Node;
 
+import com.l2jserver.gameserver.datatables.ItemTable;
 import com.l2jserver.gameserver.engines.DocumentParser;
 import com.l2jserver.gameserver.enums.IllegalActionPunishmentType;
 import com.l2jserver.gameserver.model.holders.ItemHolder;
@@ -104,6 +105,7 @@
 	public static final String CHAT_FILTER_FILE = "./config/chatfilter.txt";
 	public static final String EMAIL_CONFIG_FILE = "./config/Email.properties";
 	public static final String CH_SIEGE_FILE = "./config/ConquerableHallSiege.properties";
+	public static final String SERVICES_CONFIG_FILE = "./config/Services.properties";
 	// --------------------------------------------------
 	// L2J Variable Definitions
 	// --------------------------------------------------
@@ -1116,6 +1118,24 @@
 	public static int CHS_FAME_AMOUNT;
 	public static int CHS_FAME_FREQUENCY;
 	
+	/** Services Settings **/
+	public static boolean ALLOW_SERVICE_VOICED;
+	public static boolean PREMIUM_SERVICE_ENABLED;
+	public static boolean PREMIUM_ALLOW_VOICED;
+	public static boolean PREMIUM_PARTY_DROPSPOIL;
+	public static Map<String, ItemHolder> CHAR_PREMIUM_PRICE;
+	public static Map<String, ItemHolder> ACCOUNT_PREMIUM_PRICE;
+	public static boolean SHOW_PREMIUM_STATUS;
+	public static int NEWBIES_PREMIUM_PERIOD;
+	public static boolean NOTIFY_PREMIUM_EXPIRATION;
+	
+	public static float PREMIUM_RATE_XP;
+	public static float PREMIUM_RATE_SP;
+	public static float PREMIUM_RATE_SPOIL;
+	public static float PREMIUM_RATE_DROP_ITEMS;
+	public static float PREMIUM_RATE_DROP_ITEMS_BY_RAID;
+	public static Map<Integer, Float> PREMIUM_RATE_DROP_ITEMS_ID;
+	
 	/**
 	 * This class initializes all global variables for configuration.<br>
 	 * If the key doesn't appear in properties file, a default value is set by this class. {@link #CONFIGURATION_FILE} (properties file) for configuring your server.
@@ -2063,7 +2083,7 @@
 			RAID_MIN_RESPAWN_MULTIPLIER = NPC.getFloat("RaidMinRespawnMultiplier", 1.0f);
 			RAID_MAX_RESPAWN_MULTIPLIER = NPC.getFloat("RaidMaxRespawnMultiplier", 1.0f);
 			RAID_MINION_RESPAWN_TIMER = NPC.getInt("RaidMinionRespawnTime", 300000);
-			final String[] propertySplit = NPC.getString("CustomMinionsRespawnTime", "").split(";");
+			String[] propertySplit = NPC.getString("CustomMinionsRespawnTime", "").split(";");
 			MINIONS_RESPAWN_TIME = new HashMap<>(propertySplit.length);
 			for (String prop : propertySplit)
 			{
@@ -2762,6 +2782,141 @@
 			CHS_ENABLE_FAME = ClanHallSiege.getBoolean("EnableFame", false);
 			CHS_FAME_AMOUNT = ClanHallSiege.getInt("FameAmount", 0);
 			CHS_FAME_FREQUENCY = ClanHallSiege.getInt("FameFrequency", 0);
+			
+			// Services properties
+			final PropertiesParser ServicesProperties = new PropertiesParser(SERVICES_CONFIG_FILE);
+			
+			ALLOW_SERVICE_VOICED = ServicesProperties.getBoolean("AllowServiceVoiced", false);
+			PREMIUM_SERVICE_ENABLED = ServicesProperties.getBoolean("PremiumServiceEnabled", false);
+			PREMIUM_ALLOW_VOICED = ServicesProperties.getBoolean("PremiumAllowVoiced", false);
+			PREMIUM_PARTY_DROPSPOIL = ServicesProperties.getBoolean("PremiumPartyDropSpoil", false);
+			SHOW_PREMIUM_STATUS = ServicesProperties.getBoolean("ShowPremiumStatus", true);
+			NEWBIES_PREMIUM_PERIOD = ServicesProperties.getInt("NewbiesPremiumPeriod", 0);
+			NOTIFY_PREMIUM_EXPIRATION = ServicesProperties.getBoolean("NotifyPremiumExpiration", true);
+			
+			PREMIUM_RATE_XP = ServicesProperties.getFloat("PremiumRateXp", 1);
+			PREMIUM_RATE_SP = ServicesProperties.getFloat("PremiumRateSp", 1);
+			PREMIUM_RATE_DROP_ITEMS = ServicesProperties.getFloat("PremiumRateDropItems", 1);
+			PREMIUM_RATE_DROP_ITEMS_BY_RAID = ServicesProperties.getFloat("PremiumRateRaidDropItems", 1);
+			PREMIUM_RATE_SPOIL = ServicesProperties.getFloat("PremiumRateSpoil", 1);
+			
+			propertySplit = ServicesProperties.getString("PremiumRateDropItemsById", "").split(";");
+			PREMIUM_RATE_DROP_ITEMS_ID = new HashMap<>(propertySplit.length);
+			if (!propertySplit[0].isEmpty())
+			{
+				for (String item : propertySplit)
+				{
+					String[] itemSplit = item.split(",");
+					if (itemSplit.length != 2)
+					{
+						_log.warning(StringUtil.concat("Config.load(): invalid config property -> PremiumRateDropItemsById \"", item, "\""));
+					}
+					else
+					{
+						try
+						{
+							PREMIUM_RATE_DROP_ITEMS_ID.put(Integer.parseInt(itemSplit[0]), Float.parseFloat(itemSplit[1]));
+						}
+						catch (NumberFormatException nfe)
+						{
+							if (!item.isEmpty())
+							{
+								_log.warning(StringUtil.concat("Config.load(): invalid config property -> PremiumRateDropItemsById \"", item, "\""));
+							}
+						}
+					}
+				}
+			}
+			if (PREMIUM_RATE_DROP_ITEMS_ID.get(PcInventory.ADENA_ID) == 0f)
+			{
+				PREMIUM_RATE_DROP_ITEMS_ID.put(PcInventory.ADENA_ID, PREMIUM_RATE_DROP_ITEMS); // for Adena rate if not defined
+			}
+			
+			propertySplit = ServicesProperties.getString("CharPremiumPrice", "").split(";");
+			if (!propertySplit[0].isEmpty())
+			{
+				CHAR_PREMIUM_PRICE = new HashMap<>();
+				for (String item : propertySplit)
+				{
+					String[] itemSplit = item.split(",");
+					if (itemSplit.length != 3)
+					{
+						_log.warning(StringUtil.concat("Config.load(): invalid config property -> PremiumPrice \"", item, "\""));
+					}
+					else
+					{
+						if (Util.toMillis(itemSplit[0]) <= 0) // check for correct time period
+						{
+							_log.warning(StringUtil.concat("Config.load(): invalid time period -> PremiumPrice \"", itemSplit[0], "\""));
+						}
+						else
+						{
+							int itemId;
+							long itemCount;
+							try
+							{
+								itemId = Integer.parseInt(itemSplit[1]);
+								itemCount = Long.parseLong(itemSplit[2]);
+								if ((itemCount > 0) && (ItemTable.getInstance().getTemplate(itemId) == null)) // Check for existing item, if count is greater, than 0
+								{
+									_log.warning(StringUtil.concat("Config.load(): invalid item id -> PremiumPrice \"", itemSplit[1], "\""));
+								}
+								else
+								{
+									CHAR_PREMIUM_PRICE.put(itemSplit[0], new ItemHolder(itemId, itemCount));
+								}
+							}
+							catch (NumberFormatException nfe)
+							{
+								_log.warning(StringUtil.concat("Config.load(): invalid item parameters -> PremiumPrice \"", itemSplit[1] + ";" + itemSplit[2], "\""));
+							}
+						}
+					}
+				}
+			}
+			
+			propertySplit = ServicesProperties.getString("AccountPremiumPrice", "").split(";");
+			if (!propertySplit[0].isEmpty())
+			{
+				ACCOUNT_PREMIUM_PRICE = new HashMap<>();
+				for (String item : propertySplit)
+				{
+					String[] itemSplit = item.split(",");
+					if (itemSplit.length != 3)
+					{
+						_log.warning(StringUtil.concat("Config.load(): invalid config property -> PremiumPrice \"", item, "\""));
+					}
+					else
+					{
+						if (Util.toMillis(itemSplit[0]) <= 0) // check for correct time period
+						{
+							_log.warning(StringUtil.concat("Config.load(): invalid time period -> PremiumPrice \"", itemSplit[0], "\""));
+						}
+						else
+						{
+							int itemId;
+							long itemCount;
+							try
+							{
+								itemId = Integer.parseInt(itemSplit[1]);
+								itemCount = Long.parseLong(itemSplit[2]);
+								if ((itemCount > 0) && (ItemTable.getInstance().getTemplate(itemId) == null)) // Check for existing item, if count is greater, than 0
+								{
+									_log.warning(StringUtil.concat("Config.load(): invalid item id -> PremiumPrice \"", itemSplit[1], "\""));
+								}
+								else
+								{
+									ACCOUNT_PREMIUM_PRICE.put(itemSplit[0], new ItemHolder(itemId, itemCount));
+								}
+							}
+							catch (NumberFormatException nfe)
+							{
+								_log.warning(StringUtil.concat("Config.load(): invalid item parameters -> PremiumPrice \"", itemSplit[1] + ";" + itemSplit[2], "\""));
+							}
+						}
+					}
+				}
+			}
 		}
 		else if (Server.serverMode == Server.MODE_LOGINSERVER)
 		{
Index: java/com/l2jserver/gameserver/datatables/PremiumTable.java
===================================================================
--- java/com/l2jserver/gameserver/datatables/PremiumTable.java	(revision 0)
+++ java/com/l2jserver/gameserver/datatables/PremiumTable.java	(working copy)
@@ -0,0 +1,454 @@
+/*
+ * Copyright (C) 2004-2013 L2J Server
+ * 
+ * This file is part of L2J Server.
+ * 
+ * L2J Server is free software: you can redistribute it and/or modify
+ * it under the terms of the GNU General Public License as published by
+ * the Free Software Foundation, either version 3 of the License, or
+ * (at your option) any later version.
+ * 
+ * L2J Server is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
+ * General Public License for more details.
+ * 
+ * You should have received a copy of the GNU General Public License
+ * along with this program. If not, see <http://www.gnu.org/licenses/>.
+ */
+package com.l2jserver.gameserver.datatables;
+
+import java.sql.Connection;
+import java.sql.PreparedStatement;
+import java.sql.ResultSet;
+import java.util.Date;
+import java.util.logging.Logger;
+
+import com.l2jserver.Config;
+import com.l2jserver.L2DatabaseFactory;
+import com.l2jserver.gameserver.enums.ServiceType;
+import com.l2jserver.gameserver.instancemanager.ExpirableServicesManager;
+import com.l2jserver.gameserver.model.ExpirableService;
+import com.l2jserver.gameserver.model.actor.L2Character;
+import com.l2jserver.gameserver.model.actor.instance.L2PcInstance;
+import com.l2jserver.gameserver.model.holders.ItemHolder;
+import com.l2jserver.gameserver.network.serverpackets.ExBrPremiumState;
+import com.l2jserver.gameserver.scripting.scriptengine.listeners.player.PlayerDespawnListener;
+import com.l2jserver.gameserver.scripting.scriptengine.listeners.player.PlayerSpawnListener;
+import com.l2jserver.gameserver.util.TimeConstant;
+import com.l2jserver.gameserver.util.Util;
+
+/**
+ * This class manages storing of premium service inforamtion.
+ * @author GKR
+ */
+
+public class PremiumTable
+{
+	private static Logger _log = Logger.getLogger(PremiumTable.class.getName());
+	
+	public static float PREMIUM_BONUS_EXP = 1;
+	public static float PREMIUM_BONUS_SP = 1;
+	
+	// SQL DEFINITIONS
+	private static String LOAD_ACCOUNT_RECORD = "SELECT expiration FROM account_services WHERE charLogin = ? AND serviceName = ?";
+	private static String LOAD_CHAR_RECORD = "SELECT expiration FROM character_services WHERE charId = ? AND serviceName = ?";
+	private static String INSERT_ACCOUNT_RECORD = "REPLACE INTO account_services(charLogin, serviceName, expiration) VALUES (?,?,?)";
+	private static String INSERT_CHAR_RECORD = "REPLACE INTO character_services(charId, serviceName, expiration) VALUES (?,?,?)";
+	private static String REMOVE_RECORD = "DELETE FROM character_services WHERE charId = ? AND serviceName = ?";
+	
+	public PremiumTable()
+	{
+		if (Config.PREMIUM_SERVICE_ENABLED)
+		{
+			registerListeners();
+			
+			PREMIUM_BONUS_EXP = Math.max((Config.PREMIUM_RATE_XP / Config.RATE_XP), 1);
+			PREMIUM_BONUS_SP = Math.max((Config.PREMIUM_RATE_SP / Config.RATE_SP), 1);
+		}
+	}
+	
+	/**
+	 * Register listeners for player's enter world and logout
+	 */
+	private void registerListeners()
+	{
+		new PlayerSpawnListener()
+		{
+			/**
+			 * Send Premium State packet, to player with enabled premium service; send notification, if premium service will expire soon
+			 * @param player player to send info
+			 */
+			@Override
+			public void onPlayerLogin(L2PcInstance player)
+			{
+				if (Config.NOTIFY_PREMIUM_EXPIRATION && player.hasPremium() && !ExpirableServicesManager.getInstance().getService(ServiceType.PREMIUM, player).isUnlimited())
+				{
+					Date testDate = new Date(ExpirableServicesManager.getInstance().getService(ServiceType.PREMIUM, player).getExpirationDate());
+					if (Util.isToday(testDate))
+					{
+						player.sendMessage("Your premium will expire today");
+					}
+					else if (Util.isTomorrow(testDate))
+					{
+						player.sendMessage("Your premium will expire tomorrow");
+					}
+				}
+				
+				// Turn off status here too, because ExBrPremiumState doesn't work at logout
+				if (Config.SHOW_PREMIUM_STATUS)
+				{
+					player.sendPacket(new ExBrPremiumState(player.getObjectId(), (player.hasPremium() ? 1 : 0)));
+				}
+			}
+		};
+		
+		/**
+		 * Unregister player's premium service at logout
+		 * @param player player to send info
+		 */
+		new PlayerDespawnListener()
+		{
+			@Override
+			public void onPlayerLogout(L2PcInstance player)
+			{
+				if (player.hasPremium())
+				{
+					ExpirableServicesManager.getInstance().expireService(ServiceType.PREMIUM, player);
+				}
+			}
+		};
+	}
+	
+	/**
+	 * Load player's premium state from database
+	 * @param player player to load info
+	 */
+	public static void loadState(L2PcInstance player)
+	{
+		if (!Config.PREMIUM_SERVICE_ENABLED || (player == null))
+		{
+			return;
+		}
+		
+		long expirationDate = 0;
+		boolean isAccountBased = false;
+		
+		// Check account settings first
+		try (Connection con = L2DatabaseFactory.getInstance().getConnection();
+			PreparedStatement statement = con.prepareStatement(LOAD_ACCOUNT_RECORD))
+		{
+			statement.setString(1, player.getAccountName());
+			statement.setString(2, ServiceType.PREMIUM.toString());
+			ResultSet rset = statement.executeQuery();
+			
+			if (rset.next())
+			{
+				expirationDate = rset.getLong("expiration");
+				isAccountBased = true;
+			}
+			rset.close();
+		}
+		catch (Exception e)
+		{
+			_log.warning(PremiumTable.class.getName() + ": Error while loading account premium data for character " + player.getName() + ", account: " + player.getAccountName() + ": " + e);
+		}
+		
+		// check personal data, if account data either is not available, or expired
+		if ((expirationDate == 0) || (expirationDate < System.currentTimeMillis()))
+		{
+			try (Connection con = L2DatabaseFactory.getInstance().getConnection();
+				PreparedStatement statement = con.prepareStatement(LOAD_CHAR_RECORD))
+			{
+				statement.setInt(1, player.getObjectId());
+				statement.setString(2, ServiceType.PREMIUM.toString());
+				ResultSet rset = statement.executeQuery();
+				
+				if (rset.next())
+				{
+					expirationDate = rset.getLong("expiration");
+				}
+				rset.close();
+			}
+			catch (Exception e)
+			{
+				_log.warning(PremiumTable.class.getName() + ": Error while loading character premium data for character " + player.getName() + ": " + e);
+			}
+		}
+		
+		// Register service, if exists and isn't expired
+		if ((expirationDate < 0) || (expirationDate > System.currentTimeMillis()))
+		{
+			ExpirableServicesManager.getInstance().registerService(player, new ExpirableService(ServiceType.PREMIUM, expirationDate, isAccountBased));
+		}
+	}
+	
+	/**
+	 * Extends premium period for given time
+	 * @param player player to operate
+	 * @param millisCount time in milliseconds
+	 * @param unlimited {@code true} if service should be time-unlimited
+	 * @param isAccountBased {@code true} if service is account-based, {@code false} if service is character-based
+	 * @param store {@code true} if premium state should be stored in database, {@code false} if it will expire at logout
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	private static boolean addTime(L2PcInstance player, long millisCount, boolean unlimited, boolean isAccountBased, boolean store)
+	{
+		if (!Config.PREMIUM_SERVICE_ENABLED || (player == null))
+		{
+			return false;
+		}
+		
+		// Unlimited premium service will newer touched. There is also no need to touch parameter, if no-store is choosen for already enabled service
+		if (ExpirableServicesManager.getInstance().hasService(ServiceType.PREMIUM, player) && (!store || ExpirableServicesManager.getInstance().getService(ServiceType.PREMIUM, player).isUnlimited()))
+		{
+			return false;
+		}
+		
+		long expirationDate;
+		boolean success = !store;
+		
+		if (unlimited)
+		{
+			expirationDate = -1;
+		}
+		
+		else if (!store)
+		{
+			expirationDate = 0;
+		}
+		
+		else
+		{
+			expirationDate = ExpirableServicesManager.getInstance().hasService(ServiceType.PREMIUM, player) ? ExpirableServicesManager.getInstance().getService(ServiceType.PREMIUM, player).getExpirationDate() : 0;
+			expirationDate = Math.max(expirationDate, System.currentTimeMillis()) + millisCount;
+		}
+		
+		// Store info in database, if needed
+		if (store)
+		{
+			if (isAccountBased)
+			{
+				try (Connection con = L2DatabaseFactory.getInstance().getConnection();
+					PreparedStatement statement = con.prepareStatement(INSERT_ACCOUNT_RECORD))
+				{
+					statement.setString(1, player.getAccountName());
+					statement.setString(2, ServiceType.PREMIUM.toString());
+					statement.setLong(3, expirationDate);
+					statement.executeUpdate();
+					success = true;
+				}
+				catch (Exception e)
+				{
+					_log.warning(PremiumTable.class.getName() + ":  Could not save data for account " + player.getAccountName() + ", player " + player.getName() + ": " + e);
+				}
+			}
+			else
+			{
+				try (Connection con = L2DatabaseFactory.getInstance().getConnection();
+					PreparedStatement statement = con.prepareStatement(INSERT_CHAR_RECORD))
+				{
+					statement.setInt(1, player.getObjectId());
+					statement.setString(2, ServiceType.PREMIUM.toString());
+					statement.setLong(3, expirationDate);
+					statement.executeUpdate();
+					success = true;
+				}
+				catch (Exception e)
+				{
+					_log.warning(PremiumTable.class.getName() + ":  Could not save data for character " + player.getName() + ": " + e);
+				}
+			}
+		}
+		
+		// if store was successfull, or store doesn't required
+		if (success)
+		{
+			// register service
+			ExpirableServicesManager.getInstance().registerService(player, new ExpirableService(ServiceType.PREMIUM, expirationDate, isAccountBased));
+			
+			// Send Premium packet, if needed
+			if (Config.SHOW_PREMIUM_STATUS)
+			{
+				player.sendPacket(new ExBrPremiumState(player.getObjectId(), 1));
+			}
+			
+			// Send text message
+			player.sendMessage("Premium service is activated");
+		}
+		
+		return success;
+	}
+	
+	/**
+	 * Wrapper for {@link #addTime(L2PcInstance, long, boolean, boolean, boolean)}. Extends premium period for given time, store state in database
+	 * @param player player to operate
+	 * @param millisCount time in milliseconds
+	 * @param isAccountBased {@code true} if service is account-based, {@code false} if service is character-based
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	public static boolean addTime(L2PcInstance player, long millisCount, boolean isAccountBased)
+	{
+		return addTime(player, millisCount, false, isAccountBased, true);
+	}
+	
+	/**
+	 * Wrapper for {@link #addTime(L2PcInstance, long, boolean, boolean, boolean)}. Extends premium period for given time, store state in database
+	 * @param player player to operate
+	 * @param millisCount time in milliseconds
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	public static boolean addTime(L2PcInstance player, long millisCount)
+	{
+		if ((millisCount <= 0) || !ExpirableServicesManager.getInstance().hasService(ServiceType.PREMIUM, player))
+		{
+			return false;
+		}
+		
+		return addTime(player, millisCount, ExpirableServicesManager.getInstance().getService(ServiceType.PREMIUM, player).isAccountBased());
+	}
+	
+	/**
+	 * Wrapper for {@link #addTime(L2PcInstance, long, boolean, boolean, boolean)}. Extends premium period for given time, store state in database
+	 * @param player player to operate
+	 * @param val type of time period
+	 * @param count number of given period
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	public static boolean addTime(L2PcInstance player, TimeConstant val, int count)
+	{
+		return addTime(player, val.getTimeInMillis() * count);
+	}
+	
+	/**
+	 * Wrapper for {@link #addTime(L2PcInstance, long, boolean, boolean, boolean)}. Extends premium period for unlimited time, store state in database
+	 * @param player player to operate
+	 * @param isAccountBased {@code true} if service is account-based, {@code false} if service is character-based
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	public static boolean setUnlimitedPremium(L2PcInstance player, boolean isAccountBased)
+	{
+		return addTime(player, 0, true, isAccountBased, true);
+	}
+	
+	/**
+	 * Wrapper for {@link #addTime(L2PcInstance, long, boolean, boolean, boolean)}. Activates account premium service for player - give temporary premium, expiring at logout
+	 * @param player player to operate
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	public static boolean setTemporaryPremium(L2PcInstance player)
+	{
+		return addTime(player, 0, false, false, false);
+	}
+	
+	/**
+	 * Unregister premium service from given player and remove it from database
+	 * @param player player to operate
+	 */
+	public static void removeService(L2PcInstance player)
+	{
+		if ((player == null) || !player.hasPremium())
+		{
+			return;
+		}
+		
+		try (Connection con = L2DatabaseFactory.getInstance().getConnection();
+			PreparedStatement statement = con.prepareStatement(REMOVE_RECORD))
+		{
+			statement.setInt(1, player.getObjectId());
+			statement.setString(2, ServiceType.PREMIUM.toString());
+			statement.execute();
+		}
+		catch (Exception e)
+		{
+			_log.warning(PremiumTable.class.getName() + ":  Could not remove data for character " + player.getName() + ": " + e);
+		}
+		
+		ExpirableServicesManager.getInstance().expireService(ServiceType.PREMIUM, player);
+	}
+	
+	/**
+	 * Gets price of given premium period
+	 * @param period period to calculate
+	 * @param isAccountBased {@code true} if service is account-based, {@code false} if service is character-based
+	 * @return correspondent ItemHolder for given period and base
+	 */
+	public static ItemHolder getPrice(String period, boolean isAccountBased)
+	{
+		return isAccountBased ? Config.ACCOUNT_PREMIUM_PRICE.get(period) : Config.CHAR_PREMIUM_PRICE.get(period);
+	}
+	
+	/**
+	 * Activates premium service for player
+	 * @param player player to activate
+	 * @param period time period for activation
+	 * @param seller reference character (for logging puproses)
+	 * @param isAccountBased {@code true} if service is account-based, {@code false} if service is character-based
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	private static boolean givePremium(L2PcInstance player, String period, L2Character seller, boolean isAccountBased)
+	{
+		ItemHolder payItem = getPrice(period, isAccountBased);
+		if ((player == null) || !player.isOnline() || (payItem == null))
+		{
+			if (payItem == null)
+			{
+				_log.warning(PremiumTable.class.getName() + ":  No payitem is defined for period " + period);
+			}
+			return false;
+		}
+		
+		// do not allow player to buy another type of service (character VS account), if already has one
+		if (ExpirableServicesManager.getInstance().hasService(ServiceType.PREMIUM, player) && (ExpirableServicesManager.getInstance().getService(ServiceType.PREMIUM, player).isAccountBased() != isAccountBased))
+		{
+			player.sendMessage("Incompatible premium modes");
+		}
+		
+		else if ((payItem.getCount() > 0) && (player.getInventory().getInventoryItemCount(payItem.getId(), -1, false) < payItem.getCount()))
+		{
+			player.sendMessage("Not enough items to use");
+		}
+		
+		else if (addTime(player, Util.toMillis(period), isAccountBased))
+		{
+			player.destroyItemByItemId("Premium service", payItem.getId(), payItem.getCount(), seller, true);
+			return true;
+		}
+		
+		return false;
+	}
+	
+	/**
+	 * Wrapper for {@link #givePremium(L2PcInstance, String, L2Character, boolean)}. Activates account premium service for player
+	 * @param player player to activate
+	 * @param period time period for activation
+	 * @param seller reference character (for logging puproses)
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	public static boolean giveAccountPremium(L2PcInstance player, String period, L2Character seller)
+	{
+		return givePremium(player, period, seller, true);
+	}
+	
+	/**
+	 * Wrapper for {@link #givePremium(L2PcInstance, String, L2Character, boolean)}. Activates character premium service for player
+	 * @param player player to activate
+	 * @param period time period for activation
+	 * @param seller reference character (for logging puproses)
+	 * @return {@code true} if operation is successfull, {@code false} otherwise
+	 */
+	public static boolean giveCharPremium(L2PcInstance player, String period, L2Character seller)
+	{
+		return givePremium(player, period, seller, false);
+	}
+	
+	public static final PremiumTable getInstance()
+	{
+		return SingletonHolder._instance;
+	}
+	
+	private static class SingletonHolder
+	{
+		protected static final PremiumTable _instance = new PremiumTable();
+	}
+}
\ No newline at end of file
Index: java/com/l2jserver/gameserver/enums/Rates.java
===================================================================
--- java/com/l2jserver/gameserver/enums/Rates.java	(revision 0)
+++ java/com/l2jserver/gameserver/enums/Rates.java	(working copy)
@@ -0,0 +1,31 @@
+/*
+ * Copyright (C) 2004-2013 L2J Server
+ * 
+ * This file is part of L2J Server.
+ * 
+ * L2J Server is free software: you can redistribute it and/or modify
+ * it under the terms of the GNU General Public License as published by
+ * the Free Software Foundation, either version 3 of the License, or
+ * (at your option) any later version.
+ * 
+ * L2J Server is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
+ * General Public License for more details.
+ * 
+ * You should have received a copy of the GNU General Public License
+ * along with this program. If not, see <http://www.gnu.org/licenses/>.
+ */
+package com.l2jserver.gameserver.enums;
+
+/**
+ * Enum of some rates.
+ * @author GKR
+ */
+public enum Rates
+{
+	PREMIUM_BONUS_EXP,
+	PREMIUM_BONUS_SP,
+	SPOIL,
+	DROP_ITEM;
+}
Index: java/com/l2jserver/gameserver/enums/ServiceType.java
===================================================================
--- java/com/l2jserver/gameserver/enums/ServiceType.java	(revision 0)
+++ java/com/l2jserver/gameserver/enums/ServiceType.java	(working copy)
@@ -0,0 +1,27 @@
+/*
+ * Copyright (C) 2004-2013 L2J Server
+ * 
+ * This file is part of L2J Server.
+ * 
+ * L2J Server is free software: you can redistribute it and/or modify
+ * it under the terms of the GNU General Public License as published by
+ * the Free Software Foundation, either version 3 of the License, or
+ * (at your option) any later version.
+ * 
+ * L2J Server is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
+ * General Public License for more details.
+ * 
+ * You should have received a copy of the GNU General Public License
+ * along with this program. If not, see <http://www.gnu.org/licenses/>.
+ */
+package com.l2jserver.gameserver.enums;
+/**
+	* Contains possible service types
+	* @author GKR
+	*/
+public enum ServiceType
+{
+	PREMIUM;
+}
Index: java/com/l2jserver/gameserver/GameServer.java
===================================================================
--- java/com/l2jserver/gameserver/GameServer.java	(revision 6303)
+++ java/com/l2jserver/gameserver/GameServer.java	(working copy)
@@ -74,6 +74,7 @@
 import com.l2jserver.gameserver.datatables.OfflineTradersTable;
 import com.l2jserver.gameserver.datatables.OptionsData;
 import com.l2jserver.gameserver.datatables.PetDataTable;
+import com.l2jserver.gameserver.datatables.PremiumTable;
 import com.l2jserver.gameserver.datatables.RecipeData;
 import com.l2jserver.gameserver.datatables.SecondaryAuthData;
 import com.l2jserver.gameserver.datatables.SkillLearnData;
@@ -100,6 +101,7 @@
 import com.l2jserver.gameserver.instancemanager.CursedWeaponsManager;
 import com.l2jserver.gameserver.instancemanager.DayNightSpawnManager;
 import com.l2jserver.gameserver.instancemanager.DimensionalRiftManager;
+import com.l2jserver.gameserver.instancemanager.ExpirableServicesManager;
 import com.l2jserver.gameserver.instancemanager.FortManager;
 import com.l2jserver.gameserver.instancemanager.FortSiegeManager;
 import com.l2jserver.gameserver.instancemanager.FourSepulchersManager;
@@ -256,6 +258,8 @@
 		RaidBossPointsManager.getInstance();
 		PetDataTable.getInstance();
 		CharSummonTable.getInstance().init();
+		PremiumTable.getInstance();
+		ExpirableServicesManager.getInstance();
 		
 		printSection("Clans");
 		ClanTable.getInstance();
Index: java/com/l2jserver/gameserver/idfactory/IdFactory.java
===================================================================
--- java/com/l2jserver/gameserver/idfactory/IdFactory.java	(revision 6303)
+++ java/com/l2jserver/gameserver/idfactory/IdFactory.java	(working copy)
@@ -123,7 +123,8 @@
 	private static final String[] TIMESTAMPS_CLEAN =
 	{
 		"DELETE FROM character_instance_time WHERE time <= ?",
-		"DELETE FROM character_skills_save WHERE restore_type = 1 AND systime <= ?"
+		"DELETE FROM character_skills_save WHERE restore_type = 1 AND systime <= ?",
+		"DELETE FROM character_services WHERE expiration <= ? AND expiration >= 0"		
 	};
 	
 	protected boolean _initialized;
@@ -260,6 +261,7 @@
 			cleanCount += stmt.executeUpdate("DELETE FROM character_quest_global_data WHERE character_quest_global_data.charId NOT IN (SELECT charId FROM characters);");
 			cleanCount += stmt.executeUpdate("DELETE FROM character_tpbookmark WHERE character_tpbookmark.charId NOT IN (SELECT charId FROM characters);");
 			cleanCount += stmt.executeUpdate("DELETE FROM character_variables WHERE character_variables.charId NOT IN (SELECT charId FROM characters);");
+			cleanCount += stmt.executeUpdate("DELETE FROM character_services WHERE character_services.charId NOT IN (SELECT charId FROM characters);");
 			
 			// If the clan does not exist...
 			cleanCount += stmt.executeUpdate("DELETE FROM clan_privs WHERE clan_privs.clan_id NOT IN (SELECT clan_id FROM clan_data);");
Index: java/com/l2jserver/gameserver/instancemanager/ExpirableServicesManager.java
===================================================================
--- java/com/l2jserver/gameserver/instancemanager/ExpirableServicesManager.java	(revision 0)
+++ java/com/l2jserver/gameserver/instancemanager/ExpirableServicesManager.java	(working copy)
@@ -0,0 +1,143 @@
+/*
+ * Copyright (C) 2004-2013 L2J Server
+ * 
+ * This file is part of L2J Server.
+ * 
+ * L2J Server is free software: you can redistribute it and/or modify
+ * it under the terms of the GNU General Public License as published by
+ * the Free Software Foundation, either version 3 of the License, or
+ * (at your option) any later version.
+ * 
+ * L2J Server is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
+ * General Public License for more details.
+ * 
+ * You should have received a copy of the GNU General Public License
+ * along with this program. If not, see <http://www.gnu.org/licenses/>.
+ */
+package com.l2jserver.gameserver.instancemanager;
+
+import java.util.HashMap;
+import java.util.Map;
+
+import javolution.util.FastMap;
+
+import com.l2jserver.Config;
+import com.l2jserver.gameserver.enums.ServiceType;
+import com.l2jserver.gameserver.model.ExpirableService;
+import com.l2jserver.gameserver.model.L2World;
+import com.l2jserver.gameserver.model.actor.instance.L2PcInstance;
+import com.l2jserver.gameserver.network.serverpackets.ExBrPremiumState;
+
+/**
+ * This class manages time expirable services.
+ * @author GKR
+ */
+
+public class ExpirableServicesManager
+{
+	private final Map<ServiceType, Map<Integer, ExpirableService>> _holder; // store service maps
+	
+	protected ExpirableServicesManager()
+	{
+		_holder = new HashMap<>();
+		for (ServiceType type : ServiceType.values())
+		{
+			_holder.put(type, new FastMap<Integer, ExpirableService>().shared());
+		}
+	}
+	
+	/**
+	 * Register service for player in expiration date holder
+	 * @param player player to register
+	 * @param service type of service
+	 */
+	public void registerService(L2PcInstance player, ExpirableService service)
+	{
+		_holder.get(service.getType()).put(player.getObjectId(), service);
+	}
+	
+	public ExpirableService getService(ServiceType type, L2PcInstance player)
+	{
+		return _holder.get(type).get(player.getObjectId());
+	}
+	
+	/**
+	 * @param type type of service
+	 * @param player to check
+	 * @return {@code true} if given service type is registered for given player, {@code false} otherwise
+	 */
+	public boolean hasService(ServiceType type, L2PcInstance player)
+	{
+		return _holder.get(type).containsKey(player.getObjectId());
+	}
+	
+	/**
+	 * Send message and premium state packet to player with given id, when service of given type is expiring
+	 * @param type type of service
+	 * @param playerId objectId of player to send info
+	 */
+	private void expireService(ServiceType type, int playerId)
+	{
+		L2PcInstance player = L2World.getInstance().getPlayer(playerId);
+		if ((player != null) && player.isOnline())
+		{
+			switch (type)
+			{
+				case PREMIUM:
+				{
+					player.sendMessage("Premium service is expired");
+					if (Config.SHOW_PREMIUM_STATUS)
+					{
+						player.sendPacket(new ExBrPremiumState(player.getObjectId(), 0));
+					}
+					break;
+				}
+			}
+		}
+	}
+	
+	/**
+	 * Unregister service of given type for given player
+	 * @param type type of service
+	 * @param player to process
+	 */
+	public void expireService(ServiceType type, L2PcInstance player)
+	{
+		_holder.get(type).remove(player.getObjectId());
+		expireService(type, player.getObjectId()); // show appropriate things
+	}
+	
+	/**
+	 * Iterate over service stores, check for service expiration and unregister expired services
+	 */
+	public void checkExpiration()
+	{
+		if (Config.PREMIUM_SERVICE_ENABLED)
+		{
+			for (ServiceType type : _holder.keySet())
+			{
+				Map<Integer, ExpirableService> charMap = _holder.get(type);
+				for (int charId : charMap.keySet())
+				{
+					if (charMap.get(charId).isExpired()) // Do not touch unlimited and temporary services
+					{
+						expireService(type, charId);
+						charMap.remove(charId);
+					}
+				}
+			}
+		}
+	}
+	
+	public static final ExpirableServicesManager getInstance()
+	{
+		return SingletonHolder._instance;
+	}
+	
+	private static class SingletonHolder
+	{
+		protected static final ExpirableServicesManager _instance = new ExpirableServicesManager();
+	}
+}
\ No newline at end of file
Index: java/com/l2jserver/gameserver/model/actor/instance/L2PcInstance.java
===================================================================
--- java/com/l2jserver/gameserver/model/actor/instance/L2PcInstance.java	(revision 6303)
+++ java/com/l2jserver/gameserver/model/actor/instance/L2PcInstance.java	(working copy)
@@ -83,6 +83,7 @@
 import com.l2jserver.gameserver.datatables.ItemTable;
 import com.l2jserver.gameserver.datatables.NpcTable;
 import com.l2jserver.gameserver.datatables.PetDataTable;
+import com.l2jserver.gameserver.datatables.PremiumTable;
 import com.l2jserver.gameserver.datatables.RecipeData;
 import com.l2jserver.gameserver.datatables.SkillTable;
 import com.l2jserver.gameserver.datatables.SkillTable.FrequentSkill;
@@ -94,6 +95,8 @@
 import com.l2jserver.gameserver.enums.MountType;
 import com.l2jserver.gameserver.enums.PcRace;
 import com.l2jserver.gameserver.enums.QuestEventType;
+import com.l2jserver.gameserver.enums.Rates;
+import com.l2jserver.gameserver.enums.ServiceType;
 import com.l2jserver.gameserver.enums.Sex;
 import com.l2jserver.gameserver.enums.ShotType;
 import com.l2jserver.gameserver.handler.IItemHandler;
@@ -105,6 +108,7 @@
 import com.l2jserver.gameserver.instancemanager.CursedWeaponsManager;
 import com.l2jserver.gameserver.instancemanager.DimensionalRiftManager;
 import com.l2jserver.gameserver.instancemanager.DuelManager;
+import com.l2jserver.gameserver.instancemanager.ExpirableServicesManager;
 import com.l2jserver.gameserver.instancemanager.FortManager;
 import com.l2jserver.gameserver.instancemanager.FortSiegeManager;
 import com.l2jserver.gameserver.instancemanager.GrandBossManager;
@@ -326,6 +330,7 @@
 import com.l2jserver.gameserver.taskmanager.AttackStanceTaskManager;
 import com.l2jserver.gameserver.util.Broadcast;
 import com.l2jserver.gameserver.util.FloodProtectors;
+import com.l2jserver.gameserver.util.TimeConstant;
 import com.l2jserver.gameserver.util.Util;
 import com.l2jserver.util.Rnd;
 
@@ -936,6 +941,11 @@
 		player.setNewbie(1);
 		// Give 20 recommendations
 		player.setRecomLeft(20);
+		//Give premium to Newbie, if needed
+		if (Config.NEWBIES_PREMIUM_PERIOD > 0)
+		{
+			PremiumTable.addTime(player, TimeConstant.DAY, Config.NEWBIES_PREMIUM_PERIOD);
+		}
 		// Add the player in the characters table of the database
 		return player.createDb() ? player : null;
 	}
@@ -7377,6 +7387,7 @@
 			}
 			
 			player.restoreZoneRestartLimitTime();
+			PremiumTable.loadState(player);	// restore premium status
 			
 			if (player.isGM())
 			{
@@ -14935,4 +14946,103 @@
 	{
 		return PunishmentManager.getInstance().hasPunishment(getObjectId(), PunishmentAffect.CHARACTER, PunishmentType.PARTY_BAN);
 	}
+
+	/**
+	 * @return {@code true} if premium service is registered for player, {@code false} otherwise
+	 */
+	public boolean hasPremium()
+	{
+		return ExpirableServicesManager.getInstance().hasService(ServiceType.PREMIUM, this);
+	}
+	
+	/**
+	 * @param rateType type of rate to check
+	 * @param itemId item id to check
+	 * @param isRaid {@code true} if rate should be calculated against Raid boss
+	 * @return rate value for given rate type and item id
+	 */
+	public float getRate(Rates rateType, int itemId, boolean isRaid)
+	{
+		float rate = 0;
+		switch(rateType)
+		{
+			case PREMIUM_BONUS_EXP:
+			{
+				rate = hasPremium() ? PremiumTable.PREMIUM_BONUS_EXP : 1;
+				break;
+			}
+			case PREMIUM_BONUS_SP:
+			{
+				rate = hasPremium() ? PremiumTable.PREMIUM_BONUS_SP : 1;
+				break;
+			}
+			case SPOIL:
+			case DROP_ITEM:
+			{
+				// check for premium owner in party, if enabled by config
+				boolean hasPremium = false;
+				if (Config.PREMIUM_PARTY_DROPSPOIL && !hasPremium() && isInParty())
+				{
+					for (L2PcInstance pl : getParty().getMembers())
+					{
+						if (pl.hasPremium() && Util.checkIfInRange(Config.ALT_PARTY_RANGE, this, pl, true))
+						{
+							hasPremium = true;
+							break;
+						}
+					}
+				}
+				else
+				{
+					hasPremium = hasPremium();
+				}
+
+				if (rateType == Rates.SPOIL)
+				{
+					rate = hasPremium ? Config.PREMIUM_RATE_SPOIL : Config.RATE_DROP_SPOIL;
+				}
+				else
+				{
+					if (hasPremium)
+					{
+						if (Config.PREMIUM_RATE_DROP_ITEMS_ID.containsKey(itemId)) // check for overriden rate in premium list first
+						{
+							rate = Config.PREMIUM_RATE_DROP_ITEMS_ID.get(itemId);
+						}
+						else if (Config.RATE_DROP_ITEMS_ID.containsKey(itemId)) // then check for overriden rate in general list
+						{
+							rate = Config.RATE_DROP_ITEMS_ID.get(itemId);
+						}
+						else // return premium rate, either for raid, or normal mob, if it isn't overriden anywhere
+						{
+							rate = isRaid ? Config.PREMIUM_RATE_DROP_ITEMS_BY_RAID : Config.PREMIUM_RATE_DROP_ITEMS;
+						}
+					}
+					else
+					{
+						if (Config.RATE_DROP_ITEMS_ID.containsKey(itemId)) // check for overriden rate in general list first
+						{
+							rate = Config.RATE_DROP_ITEMS_ID.get(itemId);
+						}
+					  else // return general rate, either for raid, or normal mob, if it isn't overriden anywhere
+					  {
+							rate = isRaid ? Config.RATE_DROP_ITEMS_BY_RAID : Config.RATE_DROP_ITEMS;
+						}
+					}
+				}
+				break;
+			}
+		}
+		
+		return rate;
+	}
+	
+	/**
+	 * @param rateType type of rate to check
+	 * @return rate value for given rate type
+	 */
+	public float getRate(Rates rateType)
+	{
+		return getRate(rateType, -1, false);
+	}	
 }
\ No newline at end of file
Index: java/com/l2jserver/gameserver/model/actor/L2Attackable.java
===================================================================
--- java/com/l2jserver/gameserver/model/actor/L2Attackable.java	(revision 6303)
+++ java/com/l2jserver/gameserver/model/actor/L2Attackable.java	(working copy)
@@ -41,6 +41,7 @@
 import com.l2jserver.gameserver.datatables.ManorData;
 import com.l2jserver.gameserver.enums.InstanceType;
 import com.l2jserver.gameserver.enums.QuestEventType;
+import com.l2jserver.gameserver.enums.Rates;
 import com.l2jserver.gameserver.instancemanager.CursedWeaponsManager;
 import com.l2jserver.gameserver.instancemanager.WalkingManager;
 import com.l2jserver.gameserver.model.AbsorberInfo;
@@ -1014,7 +1015,7 @@
 				deepBlueDrop = 3;
 				if (drop.getItemId() == PcInventory.ADENA_ID)
 				{
-					deepBlueDrop *= isRaid() && !isRaidMinion() ? (int) Config.RATE_DROP_ITEMS_BY_RAID : (int) Config.RATE_DROP_ITEMS;
+					deepBlueDrop *= (int) lastAttacker.getRate(Rates.DROP_ITEM, PcInventory.ADENA_ID, (isRaid() && !isRaidMinion()));
 				}
 			}
 		}
@@ -1032,18 +1033,7 @@
 		}
 		
 		// Applies Drop rates
-		if (Config.RATE_DROP_ITEMS_ID.containsKey(drop.getItemId()))
-		{
-			dropChance *= Config.RATE_DROP_ITEMS_ID.get(drop.getItemId());
-		}
-		else if (isSweep)
-		{
-			dropChance *= Config.RATE_DROP_SPOIL;
-		}
-		else
-		{
-			dropChance *= isRaid() && !isRaidMinion() ? Config.RATE_DROP_ITEMS_BY_RAID : Config.RATE_DROP_ITEMS;
-		}
+		dropChance *= lastAttacker.getRate(isSweep ? Rates.SPOIL : Rates.DROP_ITEM, drop.getItemId(), (isRaid() && !isRaidMinion()));
 		
 		if (Config.L2JMOD_CHAMPION_ENABLE && isChampion())
 		{
@@ -1161,7 +1151,7 @@
 		}
 		
 		// Applies Drop rates
-		categoryDropChance *= isRaid() && !isRaidMinion() ? Config.RATE_DROP_ITEMS_BY_RAID : Config.RATE_DROP_ITEMS;
+		categoryDropChance *= lastAttacker.getRate(Rates.DROP_ITEM, -1, (isRaid() && !isRaidMinion()));
 		
 		if (Config.L2JMOD_CHAMPION_ENABLE && isChampion())
 		{
@@ -1195,18 +1185,8 @@
 			// chance to give a 4th time.
 			// At least 1 item will be dropped for sure. So the chance will be adjusted to 100%
 			// if smaller.
+			double dropChance = drop.getChance() * lastAttacker.getRate(Rates.DROP_ITEM, drop.getItemId(), (isRaid() && !isRaidMinion()));
 			
-			double dropChance = drop.getChance();
-			
-			if (Config.RATE_DROP_ITEMS_ID.containsKey(drop.getItemId()))
-			{
-				dropChance *= Config.RATE_DROP_ITEMS_ID.get(drop.getItemId());
-			}
-			else
-			{
-				dropChance *= isRaid() && !isRaidMinion() ? Config.RATE_DROP_ITEMS_BY_RAID : Config.RATE_DROP_ITEMS;
-			}
-			
 			if (Config.L2JMOD_CHAMPION_ENABLE && isChampion())
 			{
 				dropChance *= Config.L2JMOD_CHAMPION_REWARDS;
Index: java/com/l2jserver/gameserver/model/actor/stat/PcStat.java
===================================================================
--- java/com/l2jserver/gameserver/model/actor/stat/PcStat.java	(revision 6303)
+++ java/com/l2jserver/gameserver/model/actor/stat/PcStat.java	(working copy)
@@ -22,6 +22,7 @@
 import com.l2jserver.gameserver.datatables.ExperienceTable;
 import com.l2jserver.gameserver.datatables.NpcTable;
 import com.l2jserver.gameserver.datatables.PetDataTable;
+import com.l2jserver.gameserver.enums.Rates;
 import com.l2jserver.gameserver.model.L2PetLevelData;
 import com.l2jserver.gameserver.model.PcCondOverride;
 import com.l2jserver.gameserver.model.actor.instance.L2ClassMasterInstance;
@@ -828,6 +829,9 @@
 			bonus += (bonusExp - 1);
 		}
 		
+		// Apply premium bonus
+		bonus *= getActiveChar().getRate(Rates.PREMIUM_BONUS_EXP);
+		
 		// Check for abnormal bonuses
 		bonus = Math.max(bonus, 1);
 		bonus = Math.min(bonus, Config.MAX_BONUS_EXP);
@@ -872,6 +876,9 @@
 			bonus += (bonusSp - 1);
 		}
 		
+		// Apply premium bonus
+		bonus *= getActiveChar().getRate(Rates.PREMIUM_BONUS_SP);
+		
 		// Check for abnormal bonuses
 		bonus = Math.max(bonus, 1);
 		bonus = Math.min(bonus, Config.MAX_BONUS_SP);
Index: java/com/l2jserver/gameserver/model/ExpirableService.java
===================================================================
--- java/com/l2jserver/gameserver/model/ExpirableService.java	(revision 0)
+++ java/com/l2jserver/gameserver/model/ExpirableService.java	(working copy)
@@ -0,0 +1,79 @@
+/*
+ * Copyright (C) 2004-2013 L2J Server
+ * 
+ * This file is part of L2J Server.
+ * 
+ * L2J Server is free software: you can redistribute it and/or modify
+ * it under the terms of the GNU General Public License as published by
+ * the Free Software Foundation, either version 3 of the License, or
+ * (at your option) any later version.
+ * 
+ * L2J Server is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
+ * General Public License for more details.
+ * 
+ * You should have received a copy of the GNU General Public License
+ * along with this program. If not, see <http://www.gnu.org/licenses/>.
+ */
+package com.l2jserver.gameserver.model;
+
+import com.l2jserver.gameserver.enums.ServiceType;
+
+/**
+ * This class holds info about expirable services.
+ * @author GKR  
+ */
+public final class ExpirableService
+{
+	private ServiceType _type;
+	private long _expirationDate;
+	private boolean _isAccountBased;
+	
+	public ExpirableService(ServiceType type, long expirationDate, boolean isAccountBased)
+	{
+		_type = type;
+		_expirationDate = expirationDate;
+		_isAccountBased = isAccountBased;
+	}
+	
+	public void setExpirationDate(long date)
+	{
+		_expirationDate = date;
+	}
+	
+	public ServiceType getType()
+	{
+		return _type;
+	}
+	
+	public long getExpirationDate()
+	{
+		return _expirationDate;
+	}
+	
+	public boolean isTypeOf(ServiceType type)
+	{
+		return (type == _type);
+	}
+	
+	public boolean isExpired()
+	{
+		return ((_expirationDate > 0) && (_expirationDate <= System.currentTimeMillis()));
+	}
+	
+	public boolean isTemporary()
+	{
+		return (_expirationDate == 0);
+	}
+	
+	public boolean isUnlimited()
+	{
+		return (_expirationDate == -1);
+	}
+	
+	public boolean isAccountBased()
+	{
+		return _isAccountBased;
+	}
+}
\ No newline at end of file
Index: java/com/l2jserver/gameserver/taskmanager/TaskManager.java
===================================================================
--- java/com/l2jserver/gameserver/taskmanager/TaskManager.java	(revision 6303)
+++ java/com/l2jserver/gameserver/taskmanager/TaskManager.java	(working copy)
@@ -47,6 +47,7 @@
 import com.l2jserver.gameserver.taskmanager.tasks.TaskRaidPointsReset;
 import com.l2jserver.gameserver.taskmanager.tasks.TaskRecom;
 import com.l2jserver.gameserver.taskmanager.tasks.TaskRestart;
+import com.l2jserver.gameserver.taskmanager.tasks.TaskServiceExpire;
 import com.l2jserver.gameserver.taskmanager.tasks.TaskScript;
 import com.l2jserver.gameserver.taskmanager.tasks.TaskSevenSignsUpdate;
 import com.l2jserver.gameserver.taskmanager.tasks.TaskShutdown;
@@ -195,6 +196,7 @@
 		registerTask(new TaskRaidPointsReset());
 		registerTask(new TaskRecom());
 		registerTask(new TaskRestart());
+		registerTask(new TaskServiceExpire());
 		registerTask(new TaskScript());
 		registerTask(new TaskSevenSignsUpdate());
 		registerTask(new TaskShutdown());
Index: java/com/l2jserver/gameserver/taskmanager/tasks/TaskServiceExpire.java
===================================================================
--- java/com/l2jserver/gameserver/taskmanager/tasks/TaskServiceExpire.java	(revision 0)
+++ java/com/l2jserver/gameserver/taskmanager/tasks/TaskServiceExpire.java	(working copy)
@@ -0,0 +1,52 @@
+/*
+ * Copyright (C) 2004-2013 L2J Server
+ * 
+ * This file is part of L2J Server.
+ * 
+ * L2J Server is free software: you can redistribute it and/or modify
+ * it under the terms of the GNU General Public License as published by
+ * the Free Software Foundation, either version 3 of the License, or
+ * (at your option) any later version.
+ * 
+ * L2J Server is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
+ * General Public License for more details.
+ * 
+ * You should have received a copy of the GNU General Public License
+ * along with this program. If not, see <http://www.gnu.org/licenses/>.
+ */
+package com.l2jserver.gameserver.taskmanager.tasks;
+
+import com.l2jserver.gameserver.instancemanager.ExpirableServicesManager;
+import com.l2jserver.gameserver.taskmanager.Task;
+import com.l2jserver.gameserver.taskmanager.TaskManager;
+import com.l2jserver.gameserver.taskmanager.TaskManager.ExecutedTask;
+import com.l2jserver.gameserver.taskmanager.TaskTypes;
+
+/**
+ * @author GKR
+ */
+public class TaskServiceExpire extends Task
+{
+	public static final String NAME = "service_expire";
+	
+	@Override
+	public String getName()
+	{
+		return NAME;
+	}
+	
+	@Override
+	public void onTimeElapsed(ExecutedTask task)
+	{
+		ExpirableServicesManager.getInstance().checkExpiration();
+	}
+	
+	@Override
+	public void initializate()
+	{
+		super.initializate();
+		TaskManager.addUniqueTask(NAME, TaskTypes.TYPE_FIXED_SHEDULED, "600000", "600000", "");
+	}
+}
\ No newline at end of file
Index: java/com/l2jserver/gameserver/util/TimeConstant.java
===================================================================
--- java/com/l2jserver/gameserver/util/TimeConstant.java	(revision 0)
+++ java/com/l2jserver/gameserver/util/TimeConstant.java	(working copy)
@@ -0,0 +1,73 @@
+/*
+ * Copyright (C) 2004-2013 L2J Server
+ * 
+ * This file is part of L2J Server.
+ * 
+ * L2J Server is free software: you can redistribute it and/or modify
+ * it under the terms of the GNU General Public License as published by
+ * the Free Software Foundation, either version 3 of the License, or
+ * (at your option) any later version.
+ * 
+ * L2J Server is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
+ * General Public License for more details.
+ * 
+ * You should have received a copy of the GNU General Public License
+ * along with this program. If not, see <http://www.gnu.org/licenses/>.
+ */
+package com.l2jserver.gameserver.util;
+
+/**
+ * Hold number of milliseconds for wide-using time periods
+ * @author GKR
+ */
+
+public enum TimeConstant
+{
+	NONE(-1L, "", ""),
+	SECOND(1000L, "second", "s"),
+	MINUTE(60000L, "minute", "m"),
+	HOUR(3600000L, "hour", "h"),
+	DAY(86400000L, "day", "d"),
+	WEEK(604800000L, "week", "w"),
+	MONTH(2592000000L, "Month", "M");
+	
+	/** Count of milliseconds */
+	private final long _millis;
+	/** Mnemonic name of period */
+	private final String _name;
+	/** Short name of period */
+	private final String _shortName;
+	
+	private TimeConstant(long millis, String name, String shortName)
+	{
+		_millis = millis;
+		_name = name;
+		_shortName = shortName;
+	}
+	
+ /**
+  * @return number of millisecond in time period 
+  */
+	public long getTimeInMillis()
+	{
+		return _millis;
+	}
+	
+ /**
+  * @return mnemonic name of time period 
+  */
+	public String getName()
+	{
+		return _name;
+	}
+	
+ /**
+  * @return short name of time period 
+  */
+	public String getShortName()
+	{
+		return _shortName;
+	}
+}
\ No newline at end of file
Index: java/com/l2jserver/gameserver/util/Util.java
===================================================================
--- java/com/l2jserver/gameserver/util/Util.java	(revision 6303)
+++ java/com/l2jserver/gameserver/util/Util.java	(working copy)
@@ -24,6 +24,7 @@
 import java.text.DecimalFormatSymbols;
 import java.text.NumberFormat;
 import java.text.SimpleDateFormat;
+import java.util.Calendar;
 import java.util.Collection;
 import java.util.Date;
 import java.util.List;
@@ -491,6 +492,100 @@
 		return dateFormat.format(date.getTime());
 	}
 	
+ 	/**
+	 * @param firstDate first date to check
+	 * @param secondDate second date to check
+	 * @return {@code true} if both given dates is same day
+	 */
+	public static boolean isSameDay(Date firstDate, Date secondDate)
+	{
+		Calendar first = Calendar.getInstance();
+		Calendar second = Calendar.getInstance();
+		first.setTime(firstDate);
+		second.setTime(secondDate);
+		
+		return (first.get(Calendar.ERA) == second.get(Calendar.ERA) &&
+						first.get(Calendar.YEAR) == second.get(Calendar.YEAR) &&
+						first.get(Calendar.DAY_OF_YEAR) == second.get(Calendar.DAY_OF_YEAR));
+	}
+	
+	/**
+	 * @param date date to check
+	 * @return {@code true} if given date is tomorrow
+	 */
+	public static boolean isToday(Date date)
+	{
+		Calendar now = Calendar.getInstance();
+		Calendar test = Calendar.getInstance();
+		test.setTime(date);
+		
+		return isSameDay(now.getTime(), test.getTime());
+	}
+	
+	/**
+	 * @param date date to check
+	 * @return {@code true} if given date is today
+	 */
+	public static boolean isTomorrow(Date date)
+	{
+		Calendar now = Calendar.getInstance();
+		now.add(Calendar.DAY_OF_YEAR, 1);
+		Calendar test= Calendar.getInstance();
+		test.setTime(date);
+		
+		return isSameDay(now.getTime(), test.getTime());
+	}
+	
+	/**
+	 * @param defStr string to parse. Format is "<number><supported tag>", supported tags are "s" for seconds, "m" for minutes,
+	 * "h" for hours, "d" for days, "w" for weeks, m - for conventional "month" (30 days). Valid value for example: "25s"
+	 * @return number of milliseconds in given period, or -1 if format of period is not valid
+	 */
+	public static long toMillis(String defStr)
+	{
+		if (defStr == null)
+		{
+			return -1;
+		}
+
+		long ret = -1;
+		String toNum = defStr.substring(0, defStr.length() - 1); // Whole string, except last symbol
+		String period = defStr.substring(defStr.length() - 1, defStr.length()); // assume, that last symbol is code of time period
+		
+		TimeConstant tc = getTimeConstant(period);
+		if (tc != TimeConstant.NONE)
+		{
+			try
+			{
+				int num = Integer.parseInt(toNum);
+				ret = tc.getTimeInMillis() * num;
+			}
+			catch(NumberFormatException nfe)
+			{
+				// Do nothing
+			}
+		}
+		
+		return ret;
+	}
+
+	/**
+	 * @param defStr supported tag to parse.Supported tags are "s" for seconds, "m" for minutes,
+	 * "h" for hours, "d" for days, "w" for weeks, M - for conventional "month" (30 days).
+	 * @return TimeConstant object, corresponding to given tag, or TimeConstant.NONE for invalid tags.
+	 */
+	public static TimeConstant getTimeConstant(String defStr)
+	{
+		for (TimeConstant tc : TimeConstant.values())
+		{
+			if (tc.getShortName().equals(defStr))
+			{
+				return tc;
+			}
+		}
+		return TimeConstant.NONE;
+	}
+	
 	private static final void buildHtmlBypassCache(L2PcInstance player, HtmlActionScope scope, String html)
 	{
 		String htmlLower = html.toLowerCase(Locale.ENGLISH);
