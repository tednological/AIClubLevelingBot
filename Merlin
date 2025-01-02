import discord
from discord.ext import commands
import sqlite3
import os

intents = discord.Intents.default()
intents.members = True
intents.guilds = True
intents.messages = True
intents.message_content = True  # Required to read message content
bot = commands.Bot(command_prefix='!', intents=intents, help_command=None)

MERLIN_BOT_TOKEN = os.getenv('MERLIN_BOT_TOKEN')

# Missions now have a required_level field
missions = [
    {
        "name": "The Novice's Tome of Pythonic Whisperings",
        "description": "Learn the arcane language of Python and inscribe a simple spell.",
        "materials": [
            "https://www.youtube.com/watch?v=nCDZ1zvoZg0"
        ],
        "completion_deed": "Screenshot and present a script that prints 'Greetings, traveler!'",
        "xp_reward": 50,
        "required_level": 0  
    },
    {
        "name": "Summary Statistics of Magical Moving Pictures",
        "description": "Use your knowledge of the Pythonic script to decipher an arcane dataset.",
        "materials": [
            "https://www.youtube.com/watch?v=DkjCaAMBGWM",
            "https://www.kaggle.com/datasets/fernandogarciah24/top-1000-imdb-dataset"
        ],
        "completion_deed": "Screenshot and present a script that prints out the head and summary statistics of the \"Runtime\" column of the given dataset.",
        "xp_reward": 50,
        "required_level": 1  # Must be at least level 1
    },
    {
        "name": "Illuminating the Graphical Leylines",
        "description": "Shape mystical graphs from data using Matplotlib’s visualization spells.",
        "materials": [
            "https://matplotlib.org/stable/tutorials/introductory/quick_start.html",
            "https://www.youtube.com/watch?v=3Xc3CA655Y4"
        ],
        "completion_deed": "Produce a scatter plot and a histogram from your purified dataset.",
        "xp_reward": 75,
        "required_level": 1
    },
    {
        "name": "Scrying SQL Databases",
        "description": "Speak the arcane words of SQL to summon the proper information.",
        "materials": [
            "https://www.w3schools.com/sql/default.asp",
            "https://www.youtube.com/watch?v=kbKty5ZVKMY"
        ],
        "completion_deed": "Construct SQL statements for the following situations. You have gained access to an evil lich's databased named \'DOOM\' that contains all of the information about its minions. First, you need to need to find out how many opponents you face; return the number of entries in the MINON_ID column. Second, you need to gauge the strength of the lich's minions. Return the list of minions that have at least a 7 in the \'Power\' column. Finally, you need to find the most powerful minion of the lich. Return the minion with the highest \'Power\' in the \'DOOM\' database.",
        "xp_reward": 75,
        "required_level": 2
    },
    {
        "name": "Potion Brewing and Preprocessing",
        "description": "Recitfy the missing data from your alchemical inventory.",
        "materials": [
            "https://www.freecodecamp.org/news/data-cleaning-and-preprocessing-with-pandasbdvhj/#heading-how-to-load-the-dataset",
            "https://www.geeksforgeeks.org/data-processing-with-pandas/",
            "https://drive.google.com/file/d/1PrjT68XkRLssRE4DN03aMBwhi2VQ9XsX/view?usp=sharing"
        ],
        "completion_deed": "Complete the following tasks: 1. Impute the missing values of the PurityLevel and Quantity column with the mean or average. 2. Filter out the Spoiled ingredients. 3. Scale PurityLevel and Quantity to a range of 0 to 1 for consistent potion preparation.",
        "xp_reward": 75,
        "required_level": 2
    }
]

conn = sqlite3.connect('bot_data.db')
cursor = conn.cursor()

cursor.execute('''
CREATE TABLE IF NOT EXISTS users (
    user_id INTEGER PRIMARY KEY,
    xp INTEGER DEFAULT 0
)
''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS completed_quests (
    user_id INTEGER,
    quest_name TEXT,
    PRIMARY KEY (user_id, quest_name),
    FOREIGN KEY(user_id) REFERENCES users(user_id)
)
''')

cursor.execute('''
CREATE TABLE IF NOT EXISTS pending_quests (
    user_id INTEGER,
    quest_name TEXT,
    PRIMARY KEY (user_id, quest_name)
)
''')

conn.commit()

def get_user_xp(user_id):
    cursor.execute('SELECT xp FROM users WHERE user_id = ?', (user_id,))
    result = cursor.fetchone()
    if result is None:
        cursor.execute('INSERT INTO users (user_id, xp) VALUES (?, ?)', (user_id, 0))
        conn.commit()
        return 0
    return result[0]

def set_user_xp(user_id: int, xp: int):
    # Update or insert the user's XP. If the user doesn't exist, insert a new record.
    cursor.execute('''
        INSERT INTO users (user_id, xp) VALUES (?, ?)
        ON CONFLICT(user_id) DO UPDATE SET xp=excluded.xp
    ''', (user_id, xp))
    conn.commit()

# Simple level thresholds (adjust as needed)
level_thresholds = [
    (1, 50),
    (2, 125),
    (3, 200)
]

def get_level(xp):
    level = 0
    for lvl, threshold in level_thresholds:
        if xp >= threshold:
            level = lvl
        else:
            break
    return level

def user_completed_quests(user_id):
    cursor.execute('SELECT quest_name FROM completed_quests WHERE user_id = ?', (user_id,))
    rows = cursor.fetchall()
    return [row[0] for row in rows]

def user_pending_quests(user_id):
    cursor.execute('SELECT quest_name FROM pending_quests WHERE user_id = ?', (user_id,))
    rows = cursor.fetchall()
    return [row[0] for row in rows]

def add_pending_quest(user_id, quest_name):
    cursor.execute('INSERT INTO pending_quests (user_id, quest_name) VALUES (?, ?)', (user_id, quest_name))
    conn.commit()

def remove_pending_quest(user_id, quest_name):
    cursor.execute('DELETE FROM pending_quests WHERE user_id = ? AND quest_name = ?', (user_id, quest_name))
    conn.commit()

def add_completed_quest(user_id, quest_name):
    cursor.execute('INSERT INTO completed_quests (user_id, quest_name) VALUES (?, ?)', (user_id, quest_name))
    conn.commit()

async def send_dm_or_channel_fallback(ctx, content=None, embed=None):
    try:
        if embed:
            await ctx.author.send(embed=embed)
        else:
            await ctx.author.send(content=content)
    except discord.Forbidden:
        if embed:
            await ctx.send(embed=embed)
        else:
            await ctx.send(f"{ctx.author.mention}, I couldn’t send you a DM. Please enable DMs and try again.\n{content if content else ''}")

@bot.event
async def on_ready():
    print(f'Bot is online as {bot.user}.')

@bot.command()
async def missions_list(ctx):
    user_id = ctx.author.id
    user_xp = get_user_xp(user_id)
    user_level = get_level(user_xp)
    completed = user_completed_quests(user_id)

    embed = discord.Embed(title="Available Quests", color=discord.Color.gold())
    # Show only missions for which user meets the required level
    available_missions = [m for m in missions if user_level >= m["required_level"]]

    if not available_missions:
        await send_dm_or_channel_fallback(ctx, "You have no quests available at your current level. Keep earning XP to unlock more!")
        return

    for mission in available_missions:
        mission_name_display = mission["name"] if mission["name"] not in completed else f"~~{mission['name']}~~"
        embed.add_field(
            name=mission_name_display,
            value=f"**Description:** {mission['description']}\n**Required Level:** {mission['required_level']}\n**Reward:** {mission['xp_reward']} XP",
            inline=False
        )
    await send_dm_or_channel_fallback(ctx, embed=embed)

@bot.command()
async def mission(ctx, action: str, *, mission_name: str):
    user_id = ctx.author.id
    if mission_name is None:
        await send_dm_or_channel_fallback(ctx, "Please specify the mission name.")
        return
    
    user_xp = get_user_xp(user_id)
    user_level = get_level(user_xp)

    found_mission = None
    for m in missions:
        if m["name"].lower() == mission_name.lower():
            found_mission = m
            break
    
    if found_mission is None:
        await send_dm_or_channel_fallback(ctx, "No such mission exists. Check the mission name and try again.")
        return

    # Check if user meets level requirement
    if user_level < found_mission["required_level"]:
        await send_dm_or_channel_fallback(ctx, f"You need to be at least level {found_mission['required_level']} to attempt this quest.")
        return

    if action.lower() == "start":
        materials_text = "\n".join(found_mission["materials"])
        msg = (
            f"**{found_mission['name']}** has been started!\n"
            f"**Materials:**\n{materials_text}\n"
            f"**Completion Deed:** {found_mission['completion_deed']}"
        )
        await send_dm_or_channel_fallback(ctx, msg)

    elif action.lower() == "complete":
        completed = user_completed_quests(user_id)
        if found_mission["name"] in completed:
            await send_dm_or_channel_fallback(ctx, f"You have already completed **{found_mission['name']}**.")
            return

        pending = user_pending_quests(user_id)
        if found_mission["name"] in pending:
            await send_dm_or_channel_fallback(ctx, "You have already submitted this quest for approval. Please wait for an officer to review.")
            return

        add_pending_quest(user_id, found_mission["name"])
        await send_dm_or_channel_fallback(ctx, f"Your completion request for **{found_mission['name']}** has been submitted and is now pending officer approval.")

@bot.command()
async def xp(ctx):
    user_id = ctx.author.id
    user_current_xp = get_user_xp(user_id)
    await send_dm_or_channel_fallback(ctx, f"You currently have {user_current_xp} XP.")

@bot.command()
async def status(ctx):
    user_id = ctx.author.id
    user_current_xp = get_user_xp(user_id)
    user_level = get_level(user_current_xp)
    completed = user_completed_quests(user_id)
    pending = user_pending_quests(user_id)
    completed_list = ", ".join(completed) if completed else "None"
    pending_list = ", ".join(pending) if pending else "None"
    msg = (
        f"**Your Questing Status:**\n"
        f"**Level:** {user_level}\n"
        f"**Pending Approval:** {pending_list}\n"
        f"**Completed Quests:** {completed_list}\n"
        f"**XP:** {user_current_xp}"
    )
    await send_dm_or_channel_fallback(ctx, msg)

def mission_approve(user_id: int, mission_name: str):
    # This function completes the quest for the specified user.
    # Returns a dictionary or tuple with the outcome: XP gained, old and new levels, etc.

    # Check if the quest is pending
    pending = user_pending_quests(user_id)
    if mission_name not in pending:
        return {"success": False, "reason": "Quest not pending"}

    # Find the mission data
    found_mission = None
    for m in missions:
        if m["name"].lower() == mission_name.lower():
            found_mission = m
            break

    if not found_mission:
        return {"success": False, "reason": "Mission does not exist"}

    # Complete the quest
    remove_pending_quest(user_id, found_mission["name"])
    add_completed_quest(user_id, found_mission["name"])

    # Grant XP
    current_xp = get_user_xp(user_id)
    old_level = get_level(current_xp)
    new_xp = current_xp + found_mission["xp_reward"]
    set_user_xp(user_id, new_xp)
    new_level = get_level(new_xp)

    return {
        "success": True,
        "user_id": user_id,
        "mission_name": found_mission["name"],
        "xp_gained": found_mission["xp_reward"],
        "old_level": old_level,
        "new_level": new_level,
        "new_xp": new_xp
    }

@bot.command()
async def mission_approve_cmd(ctx, user: discord.Member, *, mission_name: str):
    # This is the command that calls the mission_approve function.
    # Make sure the author of this command is an officer or has permissions

    officer_role = discord.utils.get(ctx.guild.roles, name="Officer")
    if officer_role not in ctx.author.roles:
        await ctx.send("You do not have permission to approve missions.")
        return

    result = mission_approve(user.id, mission_name)

    if not result["success"]:
        if result["reason"] == "Quest not pending":
            await ctx.send("This user does not have that quest pending approval.")
        elif result["reason"] == "Mission does not exist":
            await ctx.send("No such mission exists.")
        return

    # Inform the officer
    await ctx.send(
        f"**{result['mission_name']}** has been approved for {user.mention}. "
        f"They gained {result['xp_gained']} XP and now have {result['new_xp']} XP."
    )

    # Notify the user
    try:
        await user.send(
            f"Your completion of **{result['mission_name']}** has been approved! "
            f"You now have {result['new_xp']} XP."
        )
    except discord.Forbidden:
        pass

    # Check for level up
    if result["new_level"] > result["old_level"]:
        new_level_role_name = f"Level {result['new_level']}"
        new_level_role = discord.utils.get(ctx.guild.roles, name=new_level_role_name)

        if new_level_role is not None:
            # Optional: Remove old level roles
            for r in user.roles:
                if r.name.startswith("Level ") and r.name != new_level_role_name:
                    await user.remove_roles(r)
            await user.add_roles(new_level_role)

        # Congratulate the user publicly
        await ctx.send(f"🎉 Congratulations {user.mention}! You have advanced to **{new_level_role_name}**!")

@bot.command()
async def roadmap(ctx):
    # Group missions by their required_level
    missions_by_level = {}
    for m in missions:
        lvl = m["required_level"]
        if lvl not in missions_by_level:
            missions_by_level[lvl] = []
        missions_by_level[lvl].append(m)

    # Sort the levels
    sorted_levels = sorted(missions_by_level.keys())

    embed = discord.Embed(title="Quest Roadmap", description="Missions by Required Level", color=discord.Color.blue())

    for lvl in sorted_levels:
        # Create a list of mission names for this level
        mission_list = "\n".join([f"- {mission['name']} (Reward: {mission['xp_reward']} XP)" for mission in missions_by_level[lvl]])
        level_title = f"Level {lvl}" if lvl > 0 else "Available at Level 0+"
        embed.add_field(name=level_title, value=mission_list, inline=False)

    await send_dm_or_channel_fallback(ctx, embed=embed)

@bot.command()
async def reset_user(ctx, user: discord.Member):
    # Check if the command issuer has the Officer role
    officer_role = discord.utils.get(ctx.guild.roles, name="Officer")
    if officer_role not in ctx.author.roles:
        await ctx.send("You do not have permission to reset user data.")
        return

    user_id = user.id

    # Clear the user's completed quests
    cursor.execute('DELETE FROM completed_quests WHERE user_id = ?', (user_id,))
    conn.commit()

    # Clear the user's pending quests
    cursor.execute('DELETE FROM pending_quests WHERE user_id = ?', (user_id,))
    conn.commit()

    # Reset the user's XP
    set_user_xp(user_id, 0)

    await ctx.send(f"All quest history and XP for {user.mention} have been reset.")


@bot.command(name='help')
async def custom_help(ctx):
    embed = discord.Embed(
        title="Merlin's Guidance",
        description="Need some help, traveler? Here are the commands you seek:",
        color=discord.Color.blue()
    )

    embed.add_field(
        name="!missions_list",
        value="Lists the missions available for your current level. 🎯",
        inline=False
    )
    embed.add_field(
        name="!mission <start|complete> <mission_name>",
        value="Start or complete a mission.\n'complete' submits it for approval. 📝",
        inline=False
    )
    embed.add_field(
        name="!xp",
        value="Shows your current XP. 📈",
        inline=False
    )
    embed.add_field(
        name="!status",
        value="Displays your current level, pending approvals, completed quests, and XP. 🏆",
        inline=False
    )
    embed.add_field(
        name="!mission_approve_cmd @User <mission_name>",
        value="(Officer-only) Approves a user's completed mission. 👑",
        inline=False
    )
    embed.add_field(
        name="!roadmap",
        value="Shows all missions organized by the level they unlock. 🗺️",
        inline=False
    )
    embed.add_field(
        name="!reset_user @User",
        value="Clears all quest history and resets XP for a user.",
        inline=False
    )
    embed.add_field(
        name="!help",
        value="Displays this very help message. ❓",
        inline=False
    )
    

    embed.set_footer(text="Merlin the Wise • Your Guide in Questing")

    try:
        await ctx.author.send(embed=embed)
    except discord.Forbidden:
        await ctx.send(f"{ctx.author.mention}, I couldn’t send you a DM. Please enable DMs and try again.")

bot.run(MERLIN_BOT_TOKEN)
