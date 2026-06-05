import discord
from discord.ext import commands
import json
import os
from datetime import datetime

# Bot setup
intents = discord.Intents.default()
intents.message_content = True
intents.dm_messages = True

bot = commands.Bot(command_prefix='!', intents=intents)

# Configuration
CONFIG_FILE = 'config.json'
TICKETS_FILE = 'tickets.json'

# Load configuration
def load_config():
    if os.path.exists(CONFIG_FILE):
        with open(CONFIG_FILE, 'r') as f:
            return json.load(f)
    return {}

# Load tickets
def load_tickets():
    if os.path.exists(TICKETS_FILE):
        with open(TICKETS_FILE, 'r') as f:
            return json.load(f)
    return {}

# Save tickets
def save_tickets(tickets):
    with open(TICKETS_FILE, 'w') as f:
        json.dump(tickets, f, indent=2)

config = load_config()

@bot.event
async def on_ready():
    print(f'{bot.user} has connected to Discord!')
    await bot.change_presence(activity=discord.Activity(type=discord.ActivityType.watching, name='for support tickets'))

@bot.event
async def on_message(message):
    # Ignore bot messages
    if message.author == bot.user:
        return
    
    # Handle DM tickets
    if isinstance(message.channel, discord.DMChannel):
        if message.author.bot:
            return
        
        tickets = load_tickets()
        user_id = str(message.author.id)
        
        # Check if user has an open ticket
        if user_id in tickets and tickets[user_id]['status'] == 'open':
            # Add message to existing ticket
            tickets[user_id]['messages'].append({
                'author': 'user',
                'content': message.content,
                'timestamp': datetime.now().isoformat()
            })
            save_tickets(tickets)
            
            # Send to support channel
            support_channel_id = config.get('support_channel_id')
            if support_channel_id:
                try:
                    support_channel = bot.get_channel(int(support_channel_id))
                    embed = discord.Embed(
                        title=f"Ticket Update - {message.author.name}#{message.author.discriminator}",
                        description=message.content,
                        color=discord.Color.blue()
                    )
                    embed.set_footer(text=f"User ID: {message.author.id}")
                    await support_channel.send(embed=embed)
                except Exception as e:
                    print(f"Error sending to support channel: {e}")
            
            await message.reply("✅ Your message has been sent to our support team!")
        else:
            # Create new ticket
            tickets[user_id] = {
                'status': 'open',
                'created_at': datetime.now().isoformat(),
                'messages': [{
                    'author': 'user',
                    'content': message.content,
                    'timestamp': datetime.now().iso](#)

