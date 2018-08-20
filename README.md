import telebot
import time
from time import time
print(time())
from telebot import types

bot = telebot.TeleBot(token)

@bot.inline_handler(func=lambda query: len(query.query) is 0)
def empty_query(query):
    hint = "Botga uzgartirishlar kiritilmoqda tez orada bot qayta ishka tushadi!"
    try:
        r = types.InlineQueryResultArticle(
                id='1',
                title="Guruxlarni nazorat qiluvchi \"Bot\"",
                description=hint,
                input_message_content=types.InputTextMessageContent(
                message_text="Botga uzgartirishlar kiritilmoqda tez orada bot qayta ishka tushadi! noqulaylillar uchun uzur")
        )
        bot.answer_inline_query(query.id, [r])
    except Exception as e:
        print(e)

GROUP_ID = goup_id  # Ваш ID группы

@bot.message_handler(func=lambda message: message.entities is not None and message.chat.id == GROUP_ID)
def delete_links(message):
    for entity in message.entities:  # Пройдёмся по всем entities в поисках ссылок
        # url - обычная ссылка, text_link - ссылка, скрытая под текстом
        if entity.type in ["url", "text_link", "я веган"]:
            bot.delete_message(message.chat.id, message.message_id)
        else:
            return

def get_language(lang_code):
    # Иногда language_code может быть None
    if not lang_code:
        return "en"
    if "-" in lang_code:
        lang_code = lang_code.split("-")[0]
    if lang_code == "ru":
        return "ru"
    else:
        return "en"

strings = {
    "ru": {
        "ro_msg": "Вам запрещено отправлять сюда сообщения в течение 1 часов."
    },
    "en": {
        "ro_msg": "Sizga 1 soat ichida yozish taqiqlangan."
    }
}

restricted_messages = ["Test" "Тест"]

# Выдаём Read-only за определённые фразы
@bot.message_handler(func=lambda message: message.text and message.text.lower() in restricted_messages and message.chat.id == GROUP_ID)
def set_ro(message):
    bot.restrict_chat_member(message.chat.id, message.from_user.id, until_date=time()+6000)
    bot.send_message(message.chat.id, strings.get(get_language(message.from_user.language_code)).get("ro_msg"),
                     reply_to_message_id=message.message_id)

if __name__ == '__main__':
    bot.polling(none_stop=True)
