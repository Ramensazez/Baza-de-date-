import json
import re
from datetime import date
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes, CallbackQueryHandler

# Token-ul botului
TOKEN = "5902895007:AAEutd-k-0hSx-XAZ8H9p4LePcjJw5Dg-H0"

# Calea către fișierul JSON
JSON_FILE = "data.json"

# Admin
ADMIN_USERNAME = "tungtungtungszahur"

# Tipurile de date acceptate
DATA_TYPES = {
    "email": "📧 Adresă de email (ex. exemplu@gmail.com)",
    "telegram": "📱 Tag Telegram (ex. @utilizator)",
    "name": "👤 Nume (ex. Ion Popescu, Francesco, 5345, fgdg)"
}

# Funcție pentru a verifica validitatea datelor
def is_valid_data(data_type, value):
    if data_type == "email":
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return re.match(pattern, value) is not None
    elif data_type == "telegram":
        pattern = r'^@[a-zA-Z0-9_]{5,}$'
        return re.match(pattern, value) is not None
    elif data_type == "name":
        return len(value.strip()) > 0
    return False

# Funcție pentru a verifica duplicate (case-insensitive)
def find_similar_entry(data_type, value, data):
    if data_type not in data:
        return None
    value_lower = value.lower()
    for existing_value, info in data[data_type].items():
        if existing_value.lower() == value_lower:
            return existing_value, info
    return None

# Funcție pentru a citi datele din JSON
def read_json():
    try:
        with open(JSON_FILE, 'r') as file:
            return json.load(file)
    except (FileNotFoundError, json.JSONDecodeError):
        return {"email": {}, "telegram": {}, "name": {}}

# Funcție pentru a salva datele în JSON
def save_json(data):
    with open(JSON_FILE, 'w') as file:
        json.dump(data, file, indent=4)

# Funcție pentru a obține meniul (dinamic pentru admin)
def get_menu_message(is_admin=False):
    keyboard = [
        [InlineKeyboardButton("📧 Adresă de email", callback_data="email")],
        [InlineKeyboardButton("📱 Tag Telegram", callback_data="telegram")],
        [InlineKeyboardButton("👤 Nume", callback_data="name")],
        [InlineKeyboardButton("🔍 Caută", callback_data="search")],
        [InlineKeyboardButton("📊 Statistici astăzi", callback_data="stats_personal")]
    ]
    if is_admin:
        keyboard.append([InlineKeyboardButton("🌍 Statistici GLOBALE", callback_data="stats_global")])

    reply_markup = InlineKeyboardMarkup(keyboard)
    welcome_message = (
        "<b>👋 Bun venit!</b>\n\n"
        "Alege ce vrei să faci:\n"
        "📧 <i>Adresă de email</i>: ex. exemplu@gmail.com\n"
        "📱 <i>Tag Telegram</i>: ex. @utilizator\n"
        "👤 <i>Nume</i>: ex. Ion Popescu, Francesco, 5345, fgdg\n"
        "🔍 <i>Caută</i>: caută o adresă, tag sau nume\n"
        "📊 <i>Statistici astăzi</i>: vezi câte ai salvat azi"
    )
    if is_admin:
        welcome_message += "\n\n<b>🌟 Admin:</b> Ai acces la <i>Statistici GLOBALE</i>"
    return welcome_message, reply_markup

# Handler pentru /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.message.from_user
    username = user.username or user.first_name
    is_admin = username.lower() == ADMIN_USERNAME.lower()
    welcome_message, reply_markup = get_menu_message(is_admin)
    await update.message.reply_text(welcome_message, parse_mode="HTML", reply_markup=reply_markup)

# Handler pentru butoane
async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    user = query.from_user
    username = user.username or user.first_name
    is_admin = username.lower() == ADMIN_USERNAME.lower()

    data = read_json()
    today = str(date.today())

    if query.data == "stats_personal":
        # 📊 STATISTICI PERSONALE
        email_count = sum(1 for info in data.get("email", {}).values() if info["date"] == today and info["username"] == username)
        telegram_count = sum(1 for info in data.get("telegram", {}).values() if info["date"] == today and info["username"] == username)
        name_count = sum(1 for info in data.get("name", {}).values() if info["date"] == today and info["username"] == username)
        total_count = email_count + telegram_count + name_count

        stats_message = (
            f"<b>📊 Statisticile tale pentru {today}</b>\n\n"
            f"📧 Adrese de email salvate: <b>{email_count}</b>\n"
            f"📱 Tag-uri Telegram salvate: <b>{telegram_count}</b>\n"
            f"👤 Nume salvate: <b>{name_count}</b>\n"
            f"🔢 <b>Total salvări azi: {total_count}</b>"
        )
        await query.message.reply_text(stats_message, parse_mode="HTML")
        await query.message.delete()

    elif query.data == "stats_global" and is_admin:
        # 🌍 STATISTICI GLOBALE
        user_stats = {}
        for dtype in DATA_TYPES.keys():
            for value, info in data.get(dtype, {}).items():
                if info["date"] == today:
                    uname = info["username"]
                    if uname not in user_stats:
                        user_stats[uname] = {"email": 0, "telegram": 0, "name": 0, "total": 0}
                    user_stats[uname][dtype] += 1
                    user_stats[uname]["total"] += 1

        if not user_stats:
            await query.message.reply_text("<b>❌ Nicio salvare azi!</b>", parse_mode="HTML")
        else:
            lines = [f"<b>🌍 Statistici GLOBALE pentru {today}</b>\n"]
            total_all = 0
            for uname, counts in sorted(user_stats.items(), key=lambda x: x[1]["total"], reverse=True):
                lines.append(
                    f"\n<b>👤 @{uname}</b>\n"
                    f"  📧 Email: <b>{counts['email']}</b>\n"
                    f"  📱 Tag: <b>{counts['telegram']}</b>\n"
                    f"  👤 Nume: <b>{counts['name']}</b>\n"
                    f"  🔢 <b>Total: {counts['total']}</b>"
                )
                total_all += counts["total"]
            lines.append(f"\n<b>🌐 Total general azi: {total_all}</b>")
            stats_message = "\n".join(lines)
            await query.message.reply_text(stats_message, parse_mode="HTML")
        await query.message.delete()

    elif query.data == "back":
        welcome_message, reply_markup = get_menu_message(is_admin)
        await query.message.reply_text(welcome_message, parse_mode="HTML", reply_markup=reply_markup)
        await query.message.delete()
        return

    else:
        # Selectare tip (email, telegram, name, search)
        context.user_data["data_type"] = query.data
        keyboard = [[InlineKeyboardButton("🔙 Înapoi", callback_data="back")]]
        reply_markup = InlineKeyboardMarkup(keyboard)

        if query.data == "search":
            await query.message.reply_text(
                "🔍 Te rog să introduci o adresă de email, un tag Telegram sau un nume pentru căutare:",
                parse_mode="HTML", reply_markup=reply_markup
            )
        else:
            await query.message.reply_text(
                f"Te rog să introduci {DATA_TYPES[query.data].split(' (')[0]}:",
                parse_mode="HTML", reply_markup=reply_markup
            )
        await query.message.delete()
        return

    # Reafișează meniul după statistici
    welcome_message, reply_markup = get_menu_message(is_admin)
    await query.message.reply_text(welcome_message, parse_mode="HTML", reply_markup=reply_markup)

# Handler pentru mesaje text
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    data_type = context.user_data.get("data_type")
    if not data_type:
        await update.message.reply_text("⚠️ Te rog să folosești comanda /start pentru a începe!", parse_mode="HTML")
        return

    value = update.message.text.strip()
    username = update.message.from_user.username or update.message.from_user.first_name

    if data_type == "search":
        data = read_json()
        found = False

        for dtype in ["email", "telegram", "name"]:
            if is_valid_data(dtype, value):
                similar_entry = find_similar_entry(dtype, value, data)
                if similar_entry:
                    existing_value, info = similar_entry
                    icon = "📧" if dtype == "email" else "📱" if dtype == "telegram" else "👤"
                    await update.message.reply_text(
                        f'<b>✅ Rezultat căutare</b>\n\n'
                        f'{icon} <b>{icon} {dtype.capitalize()}:</b> <code>{existing_value}</code>\n'
                        f'👤 <b>Adăugată de:</b> @{info["username"]}\n'
                        f'📅 <b>Data:</b> {info["date"]}',
                        parse_mode="HTML"
                    )
                    found = True
                    break

        if not found:
            await update.message.reply_text(f'<b>❌ Nu s-a găsit:</b> <code>{value}</code>', parse_mode="HTML")

    else:
        if not is_valid_data(data_type, value):
            await update.message.reply_text(f"⚠️ Te rog să introduci o {DATA_TYPES[data_type].split(' (')[0]} validă!", parse_mode="HTML")
            return

        data = read_json()
        similar_entry = find_similar_entry(data_type, value, data)
        if similar_entry:
            existing_value, info = similar_entry
            await update.message.reply_text(
                f'<b>❌ {DATA_TYPES[data_type].split(" (")[0]} {existing_value} este deja salvată de utilizatorul @{info["username"]}.</b>',
                parse_mode="HTML"
            )
        else:
            if data_type not in data:
                data[data_type] = {}
            data[data_type][value] = {
                "username": username,
                "date": str(date.today())
            }
            save_json(data)
            await update.message.reply_text(
                f'<b>✅ {DATA_TYPES[data_type].split(" (")[0]} {value} a fost salvată cu succes!</b>',
                parse_mode="HTML"
            )

    # Reset + meniu
    context.user_data["data_type"] = None
    is_admin = (update.message.from_user.username or update.message.from_user.first_name).lower() == ADMIN_USERNAME.lower()
    welcome_message, reply_markup = get_menu_message(is_admin)
    await update.message.reply_text(welcome_message, parse_mode="HTML", reply_markup=reply_markup)

# Main
def main():
    application = Application.builder().token(TOKEN).build()
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CallbackQueryHandler(button))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    print("🤖 Botul rulează...")
    application.run_polling()

if __name__ == '__main__':
    main()
