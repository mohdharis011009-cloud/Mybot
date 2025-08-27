# Telegram Bot for Background Removal
# Required Libraries:
# pip install python-telegram-bot requests

import requests
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

# Replace with your Bot Token
BOT_TOKEN = "8074083210:AAFAuUaO3A9RNxxOmLu5-fVOWRMIHwJZ4Gc"
# Replace with your Remove.bg API Key
REMOVEBG_API_KEY = "4AVpoMrDASr3xpAPKC3HVCAv"  # Jaise tumne bodfather se liya

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Hello! Send me an image and I will remove its background for you.")

async def remove_bg(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.message.photo:
        photo_file = await update.message.photo[-1].get_file()
        photo_path = "user_image.png"
        await photo_file.download_to_drive(photo_path)

        # Call remove.bg API
        with open(photo_path, 'rb') as f:
            response = requests.post(
                'https://api.remove.bg/v1.0/removebg',
                files={'image_file': f},
                data={'size': 'auto'},
                headers={'X-Api-Key': REMOVEBG_API_KEY},
            )
        if response.status_code == 200:
            with open("removed_bg.png", 'wb') as out:
                out.write(response.content)
            await update.message.reply_photo(photo=open("removed_bg.png", 'rb'), caption="Background removed ✅")
        else:
            await update.message.reply_text("Failed to remove background. Please try again.")
    else:
        await update.message.reply_text("Please send a photo!")

async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Send any photo and I will remove its background for you.")

if __name__ == '__main__':
    app = ApplicationBuilder().token(BOT_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("help", help_command))
    app.add_handler(MessageHandler(filters.PHOTO, remove_bg))

    print("Bot is running...")
    app.run_polling()
