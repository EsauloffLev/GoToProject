import telebot

token = "642787161:AAH9ETHpiLfTrxI0HjIRHDEMIqIel2RbbfE"

# Обходим блокировку с помощью прокси
telebot.apihelper.proxy = {'https': 'socks5://tvorogme:TyhoRuiGhj1874@tvorog.me:6666'}

# подключаемся к телеграму
bot = telebot.TeleBot(token=token)

state = 'waiting'
ADMIN_ID = 560733832
users = set()
bets_right = {}
bets_left = {}
sum_left = 0
sum_right = 0
who_win = 0
id_bets_left = set()
id_bets_right = set()
bet_amount = 0

left_player = ""
right_player = ""

@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(message.chat.id, "HEllo")
    if message.chat.id != ADMIN_ID:
        users.add(message.chat.id)
    else:
        bot.send_message(message.chat.id, "Вы дядя админ")

# content_types=['text'] - сработает, если нам прислали текстовое сообщение
@bot.message_handler(content_types=['text'])
def process(message):
    global state
    global users
    global bets_left
    global left_player
    global right_player
    global bets_right
    global sum_left
    global sum_right
    global bet_amount
    global id_bets_lest
    global id_bets_right
    if message.chat.id == ADMIN_ID:
        if state == 'waiting':
# TODO: понять из сообщения кто сражается
            left_player = ""
            right_player = ""
            action_name = ""
            list = message.text.split('-')
            if len(list) >= 2:
                our_text = list[2] + ": " + list[0] + " vs " + list[1]
            for user in users:
# TODO: ....
                bot.send_message(user, our_text)
            state = 'taking'
        if state == 'taking':
            state = 'playing'
        if state == 'playing':
            state = 'waiting'
            if message == '1':
                who_win = 1
            elif message == '2':
                who_win = 2
            else:
                bot.send_message(message.chat.id, "Пожалуйста, напишите результат матча")
#TODO: Разослать всем, кто че выиграл
            for user in users:
                if who_win == 1:
                    if user in id_bets_left:
                        win_bet = str(bets_left[user] * (sum_left + sum_right) / sum_left)
                        result = "Поздравляем, Ваш выигрыш составляет " + win_bet
                        bot.send_message(user, result)
                    else:
                        bot.send_message(user, "В следующий раз повезет")
                elif who_win == 2:
                    if user in id_bets_right:
                        win_bet = str(bets_right[user] * (sum_left + sum_right) / sum_right)
                        result = "Поздравляем, Ваш выигрыш составляет " + win_bet
                        bot.send_message(user, result)
                    else:
                        bot.send_message(user, "В сдедующий раз повезет")
    else:
        if state == 'waiting':
            bot.send_message(message.chat.id, "Пока ждем")
        if state == 'taking':
#TODO: сделать ставку
            if message == '1':
                bot.send_message(message.chat.id, "Введите размер вашей ставки")
                bet_amount = int(message)
                sum_left+= bet_amount
                bets_right[message.chat.id] = bet_amount
                id_bets_left.add(message.chat.id)
            elif message == '2':
                bot.send_message(message.chat.id, "Введите размер вашей ставки")
                bet_amount = int(message)
                sum_right+= bet_amount
                bets_left[message.chat.id] = bet_amount
                id_bets_right.add(message.chat.id)
            else:
                bot.send_message(message.chat.id, "Sorry, Я Вас не понимать")
                bot.send_message(message.chat.id, "Ставка принята")
        if state == 'playing':
            bot.send_message(message.chat.id, "Игра идет")


# поллинг - вечный цикл с обновлением входящих сообщений
bot.polling(none_stop=True)
