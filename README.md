# Pyq-telegram-bot
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, ContextTypes, filters
import openai
import os

# 🔐 KEYS (use Render environment variables)
BOT_TOKEN = os.getenv("BOT_TOKEN")
OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
openai.api_key = OPENAI_API_KEY

def detect_subject(text):
    t = text.lower()
    if any(x in t for x in ["array", "stack", "queue", "tree", "graph"]):
        return "DSA"
    if any(x in t for x in ["program", "output", "python", "java", "c program"]):
        return "Programming"
    if any(x in t for x in ["integrate", "matrix", "limit", "probability"]):
        return "Maths"
    return "Aptitude"

def make_prompt(subject, question):
    prompts = {
        "Maths": f"Solve step by step in Hinglish.\nQuestion: {question}",
        "Programming": f"Explain code and output in Hinglish.\nQuestion: {question}",
        "DSA": f"Explain approach, algorithm, code and complexity in Hinglish.\nQuestion: {question}",
        "Aptitude": f"Solve with tricks in Hinglish.\nQuestion: {question}"
    }
    return prompts[subject]

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "👋 Welcome to PYQ Solver AI 🤖\nSend any Maths / Programming / DSA / Aptitude question."
    )

async def solve(update: Update, context: ContextTypes.DEFAULT_TYPE):
    q = update.message.text
    subject = detect_subject(q)
    prompt = make_prompt(subject, q)

    await update.message.reply_text("⏳ Solving...")

    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}]
    )

    answer = response.choices[0].message.content

    await update.message.reply_text(f"📘 Subject: {subject}\n\n{answer}")

app = ApplicationBuilder().token(BOT_TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, solve))

print("Bot started")
app.run_polling()
