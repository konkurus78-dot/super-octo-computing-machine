import telebot
from telebot import types
import json
import os

TOKEN = "8585971078:AAGhCi4oatPk8Imr2DP1YwcLqu455lojFdY"
SUPER_ADMIN = 6138428310
DB_FILE = "database.json"

# --- Database ---
def load_db():
    if not os.path.exists(DB_FILE):
        db = {
            "users": {},
            "channels": ["@kanal1", "@kanal2"],
            "admins": [SUPER_ADMIN],
            "ref_price": 5,
            "min_withdraw": 20,
            "support": "@abdullayvku",
            "withdraw_requests": []
        }
        save_db(db)
    else:
        with open(DB_FILE, "r") as f:
            db = json.load(f)
    for key in ["support","ref_price","min_withdraw","admins"]:
        if key not in db:
            if key=="support": db[key]="@abdullayvku"
            if key=="ref_price": db[key]=5
            if key=="min_withdraw": db[key]=20
            if key=="admins": db[key]=[SUPER_ADMIN]
    save_db(db)
    return db

def save_db(db):
    with open(DB_FILE,"w") as f:
        json.dump(db,f,indent=4)

db = load_db()
bot = telebot.TeleBot(TOKEN)

# --- Helpers ---
def check_user(uid):
    uid = str(uid)
    if uid not in db["users"]:
        db["users"][uid] = {"stars":0,"invited":[],"joined":False}
        save_db(db)

def is_subscribed(uid):
    if len(db["channels"])==0:
        return True
    for ch in db["channels"]:
        try:
            status = bot.get_chat_member(ch,uid).status
            if status not in ["member","administrator","creator"]:
                return False
        except:
            return False
    return True

def ref_link(uid):
    return f"https://t.me/{bot.get_me().username}?start={uid}"

def main_menu(uid):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("⭐ Stars ishlash","💳 Stars yechish")
    markup.add("📊 Hisobim","🛠 Support")
    if int(uid) in db["admins"]:
        markup.add("🛠 Admin panel")
    bot.send_message(uid,"🏠 Asosiy menyu:",reply_markup=markup)

# --- Start ---
@bot.message_handler(commands=['start'])
def start(message):
    uid = str(message.chat.id)
    check_user(uid)
    user = db["users"][uid]

    args = message.text.split()
    if len(args)==2 and not user["joined"]:
        rid = args[1]
        if rid.isdigit() and rid!=uid:
            check_user(rid)
            if uid not in db["users"][rid]["invited"]:
                db["users"][rid]["invited"].append(uid)
                db["users"][rid]["stars"] += db["ref_price"]
                save_db(db)
                bot.send_message(int(rid),f"🎉 Yangi referal! +{db['ref_price']} ⭐")
    user["joined"]=True
    save_db(db)

    # --- Adminlar uchun kanal tekshiruvi o‘tkazilmaydi ---
    if int(uid) not in db["admins"]:
        if not is_subscribed(message.chat.id):
            msg = "❗ Botdan foydalanish uchun quyidagi kanallarga obuna bo‘ling:\n"
            for ch in db["channels"]:
                msg += f"{ch}\n"
            bot.send_message(message.chat.id,msg)
            return

    main_menu(message.chat.id)

# --- User menu ---
@bot.message_handler(func=lambda m: True)
def user_menu(message):
    uid = str(message.chat.id)
    check_user(uid)
    text = message.text

    if int(uid) in db["admins"]:
        admin_menu_handler(message)
        return

    if text=="⭐ Stars ishlash":
        if not is_subscribed(message.chat.id) and int(uid) not in db["admins"]:
            bot.send_message(uid,"❗ Kanalga obuna bo‘ling, keyin referal olishingiz mumkin.")
            return
        bot.send_message(uid,f"👥 Referal link:\n{ref_link(uid)}\n\n💰 Har referal: {db.get('ref_price',5)} ⭐")

    elif text=="📊 Hisobim":
        u=db["users"][uid]
        bot.send_message(uid,f"🆔 ID: {uid}\n⭐ Stars: {u['stars']}\n👥 Referallar: {len(u['invited'])}\n🔗 Link: {ref_link(uid)}")

    elif text=="💳 Stars yechish":
        if db["users"][uid]["stars"]<db.get("min_withdraw",20):
            bot.send_message(uid,f"❗ Minimal yechish: {db.get('min_withdraw',20)} ⭐\nSizda: {db['users'][uid]['stars']} ⭐")
        else:
            msg=bot.send_message(uid,"💳 Karta yoki hisob raqamingizni kiriting:")
            bot.register_next_step_handler(msg,withdraw)

    elif text=="🛠 Support":
        bot.send_message(uid,f"📞 Support: {db.get('support','@abdullayvku')}")

    elif text=="🛠 Admin panel":
        admin_menu_handler(message)

# --- Withdraw ---
def withdraw(message):
    uid=str(message.chat.id)
    card=message.text
    stars=db["users"][uid]["stars"]
    db["users"][uid]["stars"]=0
    save_db(db)
    bot.send_message(uid,"✅ Ariza qabul qilindi!")
    bot.send_message(SUPER_ADMIN,f"📥 Yangi yechish arizasi\nID: {uid}\nStars: {stars}\nKarta: {card}")

# --- Admin menu handler ---
def admin_menu_handler(message):
    uid=message.chat.id
    text=message.text
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("➕ Admin qo‘shish","➖ Admin o‘chirish")
    markup.add("➕ Kanal qo‘shish","➖ Kanal o‘chirish")
    markup.add("⚙ Referal narxi","⚙ Minimal yechish")
    markup.add("⚙ Support sozlash","📜 Kanal ro‘yxati")
    markup.add("⬅️ Orqaga")

    if text=="🛠 Admin panel":
        bot.send_message(uid,"🛠 Admin panel:",reply_markup=markup)
    elif text=="➕ Admin qo‘shish":
        msg=bot.send_message(uid,"Admin ID kiriting:")
        bot.register_next_step_handler(msg,add_admin)
    elif text=="➖ Admin o‘chirish":
        msg=bot.send_message(uid,"O‘chiriladigan admin ID:")
        bot.register_next_step_handler(msg,del_admin)
    elif text=="➕ Kanal qo‘shish":
        msg=bot.send_message(uid,"Kanal (@username):")
        bot.register_next_step_handler(msg,add_channel)
    elif text=="➖ Kanal o‘chirish":
        msg=bot.send_message(uid,"O‘chiriladigan kanal:")
        bot.register_next_step_handler(msg,del_channel)
    elif text=="⚙ Referal narxi":
        msg=bot.send_message(uid,"Yangi referal narxi:")
        bot.register_next_step_handler(msg,change_ref_price)
    elif text=="⚙ Minimal yechish":
        msg=bot.send_message(uid,"Minimal yechish:")
        bot.register_next_step_handler(msg,change_min_withdraw)
    elif text=="⚙ Support sozlash":
        msg=bot.send_message(uid,"Yangi support username:")
        bot.register_next_step_handler(msg,change_support)
    elif text=="📜 Kanal ro‘yxati":
        ch="\n".join(db["channels"]) or "Kanal yo‘q"
        bot.send_message(uid,f"📋 Kanallar:\n{ch}")
    elif text=="⬅️ Orqaga":
        main_menu(uid)

# --- Admin functions ---
def add_admin(msg):
    try:
        new=int(msg.text)
        if new not in db["admins"]:
            db["admins"].append(new)
            save_db(db)
            bot.send_message(msg.chat.id,"Admin qo‘shildi!")
        else:
            bot.send_message(msg.chat.id,"Admin allaqachon bor!")
    except:
        bot.send_message(msg.chat.id,"ID xato!")

def del_admin(msg):
    try:
        rem=int(msg.text)
        if rem==SUPER_ADMIN:
            bot.send_message(msg.chat.id,"SUPER ADMIN o‘chirib bo‘lmaydi!")
            return
        if rem in db["admins"]:
            db["admins"].remove(rem)
            save_db(db)
            bot.send_message(msg.chat.id,"Admin o‘chirildi!")
        else:
            bot.send_message(msg.chat.id,"Admin topilmadi!")
    except:
        bot.send_message(msg.chat.id,"ID xato!")

def add_channel(msg):
    ch=msg.text
    if not ch.startswith("@"): ch="@"+ch
    if ch not in db["channels"]:
        db["channels"].append(ch)
        save_db(db)
        bot.send_message(msg.chat.id,"Kanal qo‘shildi!")
    else:
        bot.send_message(msg.chat.id,"Kanal allaqachon mavjud!")

def del_channel(msg):
    ch=msg.text
    if not ch.startswith("@"): ch="@"+ch
    if ch in db["channels"]:
        db["channels"].remove(ch)
        save_db(db)
        bot.send_message(msg.chat.id,"Kanal o‘chirildi!")
    else:
        bot.send_message(msg.chat.id,"Kanal topilmadi!")

def change_ref_price(msg):
    try:
        db["ref_price"]=int(msg.text)
        save_db(db)
        bot.send_message(msg.chat.id,"Referal narxi yangilandi!")
    except:
        bot.send_message(msg.chat.id,"Xato qiymat!")

def change_min_withdraw(msg):
    try:
        db["min_withdraw"]=int(msg.text)
        save_db(db)
        bot.send_message(msg.chat.id,"Minimal yechish yangilandi!")
    except:
        bot.send_message(msg.chat.id,"Xato qiymat!")

def change_support(msg):
    db["support"]=msg.text.strip()
    save_db(db)
    bot.send_message(msg.chat.id,"Support yangilandi!")

# --- Run ---
print("Bot ishlayapti...")
bot.infinity_polling()
