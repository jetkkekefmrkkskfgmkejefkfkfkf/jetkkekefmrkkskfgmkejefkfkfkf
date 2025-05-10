import os
import yt_dlp
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

BOT_TOKEN = "7583416374:AAF2OsAiBX2Ja_mQ5-Smqd7LOjHzH3-zLB0

# YouTube'dan ses indirme fonksiyonu
def download_audio(query):
    ydl_opts = {
        'format': 'bestaudio/best',
        'noplaylist': True,
        'quiet': True,
        'outtmpl': '%(title)s.%(ext)s',
        'postprocessors': [{
            'key': 'FFmpegExtractAudio',
            'preferredcodec': 'mp3',
            'preferredquality': '192',
        }],
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(f"ytsearch:{query}", download=True)
        title = ydl.prepare_filename(info['entries'][0])
        mp3_file = title.replace(".webm", ".mp3").replace(".m4a", ".mp3")
        return mp3_file

# /play komutu
async def play(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        await update.message.reply_text("Lütfen bir şarkı adı girin. Örn: /play sevdanın yolları")
        return

    query = " ".join(context.args)
    await update.message.reply_text(f"'{query}' aranıyor...")

    try:
        mp3_path = download_audio(query)
        await update.message.reply_audio(audio=open(mp3_path, 'rb'))
        os.remove(mp3_path)
    except Exception as e:
        await update.message.reply_text(f"Bir hata oluştu: {e}")

# Botu başlat
if name == "main":
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("play", play))
    app.run_pollin
