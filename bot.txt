#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Bot Telegram pour un quiz sur les régions de la Fédération de Russie.
"""

import logging
import random
import os
import asyncio  # <-- AJOUTÉ pour la pause

from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update
from telegram.ext import (
    Application,
    CommandHandler,
    CallbackQueryHandler,
    ConversationHandler,
    MessageHandler,
    filters,
    CallbackContext,
)

# --- Configuration ---

# On charge la clé depuis les variables d'environnement (PLUS SÉCURISÉ)
# <-- MODIFIÉ : Méthode sécurisée pour la clé API
TELEGRAM_API_KEY = os.getenv("TELEGRAM_API_KEY") 
if not TELEGRAM_API_KEY:
    # Si la variable n'est pas trouvée, on lève une erreur claire.
    # N'écrivez PAS votre clé ici. Définissez-la dans le terminal avant de lancer le script.
    raise ValueError("La variable d'environnement TELEGRAM_API_KEY n'est pas définie !")

# Configuration du logging
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO
)
logger = logging.getLogger(__name__)

# --- Données du quiz ---
REGIONS_DATA = [
    {'numero': '01', 'nom': 'Республика Адыгея', 'ville': 'Майкоп'},
    {'numero': '02', 'nom': 'Республика Башкортостан', 'ville': 'Уфа'},
    {'numero': '03', 'nom': 'Республика Бурятия', 'ville': 'Улан-Удэ'},
    {'numero': '04', 'nom': 'Республика Алтай', 'ville': 'Горно-Алтайск'},
    {'numero': '05', 'nom': 'Республика Дагестан', 'ville': 'Махачкала'},
    {'numero': '06', 'nom': 'Республика Ингушетия', 'ville': 'Магас'},
    {'numero': '07', 'nom': 'Кабардино-Балкарская Республика', 'ville': 'Нальчик'},
    {'numero': '08', 'nom': 'Республика Калмыкия', 'ville': 'Элиста'},
    {'numero': '09', 'nom': 'Карачаево-Черкесская Республика', 'ville': 'Черкесск'},
    {'numero': '10', 'nom': 'Республика Карелия', 'ville': 'Петрозаводск'},
    {'numero': '11', 'nom': 'Республика Коми', 'ville': 'Сыктывкар'},
    {'numero': '12', 'nom': 'Республика Марий Эл', 'ville': 'Йошкар-Ола'},
    {'numero': '13', 'nom': 'Республика Мордовия', 'ville': 'Саранск'},
    {'numero': '14', 'nom': 'Республика Саха (Якутия)', 'ville': 'Якутск'},
    {'numero': '15', 'nom': 'Республика Северная Осетия — Алания', 'ville': 'Владикавказ'},
    {'numero': '16', 'nom': 'Республика Татарстан', 'ville': 'Казань'},
    {'numero': '17', 'nom': 'Республика Тыва', 'ville': 'Кызыл'},
    {'numero': '18', 'nom': 'Удмуртская Республика', 'ville': 'Ижевск'},
    {'numero': '19', 'nom': 'Республика Хакасия', 'ville': 'Абакан'},
    {'numero': '20', 'nom': 'Чеченская Республика', 'ville': 'Грозный'},
    {'numero': '21', 'nom': 'Чувашская Республика', 'ville': 'Чебоксары'},
    {'numero': '22', 'nom': 'Алтайский край', 'ville': 'Барнаул'},
    {'numero': '23', 'nom': 'Краснодарский край', 'ville': 'Краснодар'},
    {'numero': '24', 'nom': 'Красноярский край', 'ville': 'Красноярск'},
    {'numero': '25', 'nom': 'Приморский край', 'ville': 'Владивосток'},
    {'numero': '26', 'nom': 'Ставропольский край', 'ville': 'Ставрополь'},
    {'numero': '27', 'nom': 'Хабаровский край', 'ville': 'Хабаровск'},
    {'numero': '28', 'nom': 'Амурская область', 'ville': 'Благовещенск'},
    {'numero': '29', 'nom': 'Архангельская область', 'ville': 'Архангельск'},
    {'numero': '30', 'nom': 'Астраханская область', 'ville': 'Астрахань'},
    {'numero': '31', 'nom': 'Белгородская область', 'ville': 'Белгород'},
    {'numero': '32', 'nom': 'Брянская область', 'ville': 'Брянск'},
    {'numero': '33', 'nom': 'Владимирская область', 'ville': 'Владимир'},
    {'numero': '34', 'nom': 'Волгоградская область', 'ville': 'Волгоград'},
    {'numero': '35', 'nom': 'Вологодская область', 'ville': 'Вологда'},
    {'numero': '36', 'nom': 'Воронежская область', 'ville': 'Воронеж'},
    {'numero': '37', 'nom': 'Ивановская область', 'ville': 'Иваново'},
    {'numero': '38', 'nom': 'Иркутская область', 'ville': 'Иркутск'},
    {'numero': '39', 'nom': 'Калининградская область', 'ville': 'Калининград'},
    {'numero': '40', 'nom': 'Калужская область', 'ville': 'Калуга'},
    {'numero': '41', 'nom': 'Камчатский край', 'ville': 'Петропавловск-Камчатский'},
    {'numero': '42', 'nom': 'Кемеровская область', 'ville': 'Кемерово'},
    {'numero': '43', 'nom': 'Кировская область', 'ville': 'Киров'},
    {'numero': '44', 'nom': 'Костромская область', 'ville': 'Кострома'},
    {'numero': '45', 'nom': 'Курганская область', 'ville': 'Курган'},
    {'numero': '46', 'nom': 'Курская область', 'ville': 'Курск'},
    {'numero': '47', 'nom': 'Ленинградская область', 'ville': 'Санкт-Петербург'},
    {'numero': '48', 'nom': 'Липецкая область', 'ville': 'Липецк'},
    {'numero': '49', 'nom': 'Магаданская область', 'ville': 'Магадан'},
    {'numero': '50', 'nom': 'Московская область', 'ville': 'Москва'},
    {'numero': '51', 'nom': 'Мурманская область', 'ville': 'Мурманск'},
    {'numero': '52', 'nom': 'Нижегородская область', 'ville': 'Нижний Новгород'},
    {'numero': '53', 'nom': 'Новгородская область', 'ville': 'Великий Новгород'},
    {'numero': '54', 'nom': 'Новосибирская область', 'ville': 'Новосибирск'},
    {'numero': '55', 'nom': 'Омская область', 'ville': 'Омск'},
    {'numero': '56', 'nom': 'Оренбургская область', 'ville': 'Оренбург'},
    {'numero': '57', 'nom': 'Орловская область', 'ville': 'Орёл'},
    {'numero': '58', 'nom': 'Пензенская область', 'ville': 'Пенза'},
    {'numero': '59', 'nom': 'Пермский край', 'ville': 'Пермь'},
    {'numero': '60', 'nom': 'Псковская область', 'ville': 'Псков'},
    {'numero': '61', 'nom': 'Ростовская область', 'ville': 'Ростов-на-Дону'},
    {'numero': '62', 'nom': 'Рязанская область', 'ville': 'Рязань'},
    {'numero': '63', 'nom': 'Самарская область', 'ville': 'Самара'},
    {'numero': '64', 'nom': 'Саратовская область', 'ville': 'Саратов'},
    {'numero': '65', 'nom': 'Сахалинская область', 'ville': 'Южно-Сахалинск'},
    {'numero': '66', 'nom': 'Свердловская область', 'ville': 'Екатеринбург'},
    {'numero': '67', 'nom': 'Смоленская область', 'ville': 'Смоленск'},
    {'numero': '68', 'nom': 'Тамбовская область', 'ville': 'Тамбов'},
    {'numero': '69', 'nom': 'Тверская область', 'ville': 'Тверь'},
    {'numero': '70', 'nom': 'Томская область', 'ville': 'Томск'},
    {'numero': '71', 'nom': 'Тульская область', 'ville': 'Тула'},
    {'numero': '72', 'nom': 'Тюменская область', 'ville': 'Тюмень'},
    {'numero': '73', 'nom': 'Ульяновская область', 'ville': 'Ульяновск'},
    {'numero': '74', 'nom': 'Челябинская область', 'ville': 'Челябинск'},
    {'numero': '75', 'nom': 'Забайкальский край', 'ville': 'Чита'},
    {'numero': '76', 'nom': 'Ярославская область', 'ville': 'Ярославль'},
    {'numero': '77', 'nom': 'город Москва', 'ville': 'Москва'},
    {'numero': '78', 'nom': 'город Санкт-Петербург', 'ville': 'Санкт-Петербург'},
    {'numero': '79', 'nom': 'Еврейская автономная область', 'ville': 'Биробиджан'},
    {'numero': '83', 'nom': 'Ненецкий автономный округ', 'ville': 'Нарьян-Мар'},
    {'numero': '86', 'nom': 'Ханты-Мансийский автономный округ - Югра', 'ville': 'Ханты-Мансийск'},
    {'numero': '87', 'nom': 'Чукотский автономный округ', 'ville': 'Анадырь'},
    {'numero': '89', 'nom': 'Ямало-Ненецкий автономный округ', 'ville': 'Салехард'},
]

# --- États de la conversation ---
SELECTING_MODE, SELECTING_TYPE, IN_QUIZ, DISCOVERY_MODE = range(4)
BEST_SCORE_KEY = 'best_survival_score'

# --- Fonctions auxiliaires ---

def get_main_menu_keyboard():
    """Retourne le clavier du menu principal."""
    keyboard = [
        [InlineKeyboardButton("🎓 Режим тренировки", callback_data='mode_training')],
        [InlineKeyboardButton("🎯 Режим 'Вызов' (10 вопросов)", callback_data='mode_challenge')],
        [InlineKeyboardButton("🏃‍♀️ Режим 'Марафон' (89 вопросов)", callback_data='mode_marathon')],
        [InlineKeyboardButton("☠️ Режим 'Выживание'", callback_data='mode_survival')],
        [InlineKeyboardButton("📚 Режим 'Справочник'", callback_data='mode_discovery')],
    ]
    return InlineKeyboardMarkup(keyboard)

def get_question_type_keyboard():
    """Retourne le clavier pour choisir le type de question."""
    keyboard = [
        [InlineKeyboardButton("Угадать столицу по региону", callback_data='type_capital')],
        [InlineKeyboardButton("Угадать регион по столице", callback_data='type_region')],
        [InlineKeyboardButton("Угадать регион по номеру", callback_data='type_number')],
        [InlineKeyboardButton("↩️ Назад в главное меню", callback_data='back_to_menu')],
    ]
    return InlineKeyboardMarkup(keyboard)

def generate_question_text(context: CallbackContext) -> str:
    """Génère le texte de la question actuelle."""
    user_data = context.user_data
    question_data = user_data['current_question']
    question_type = user_data['question_type_for_question']
    score = user_data.get('score', 0)
    
    mode_text = {
        'mode_training': "Тренировка",
        'mode_challenge': f"Вызов (Вопрос {user_data.get('question_count', 0)}/10)",
        'mode_marathon': f"Марафон (Вопрос {user_data.get('question_count', 0)}/{len(REGIONS_DATA)})",
        'mode_survival': "Выживание"
    }.get(user_data['mode'], "")

    text = f"🕹️ <b>Режим: {mode_text}</b> | 🎯 <b>Счёт: {score}</b>\n\n"

    if question_type == 'type_capital':
        text += f"Какая столица у региона: <b>{question_data['nom']} ({question_data['numero']})</b>?"
    elif question_type == 'type_region':
        text += f"Какому региону соответствует столица: <b>{question_data['ville']}</b>?"
    else: # type_number
        text += f"Какому региону соответствует номер: <b>{question_data['numero']}</b>?"
        
    return text

# --- Fonctions principales de la conversation ---

async def start(update: Update, context: CallbackContext) -> int:
    """Envoie le message d'accueil et le menu principal."""
    user = update.effective_user
    query = update.callback_query
    
    # Gestion du meilleur score
    best_score = context.bot_data.get(BEST_SCORE_KEY, 0)
    
    # <-- MODIFIÉ : Texte en russe et parse_mode='HTML'
    welcome_message = (
        f"🇷🇺 <b>Добро пожаловать, {user.first_name}!</b> 🇷🇺\n\n"
        "Готовы проверить свои знания о субъектах Российской Федерации?\n\n"
        "Выберите режим игры, чтобы начать.\n\n"
        f"🏆 <i>Лучший счёт (Выживание): {best_score}</i>"
    )

    keyboard = get_main_menu_keyboard()

    if query:
        await query.answer()
        await query.edit_message_text(
            text=welcome_message,
            reply_markup=keyboard,
            parse_mode='HTML'
        )
    else:
        await update.message.reply_text(
            text=welcome_message,
            reply_markup=keyboard,
            parse_mode='HTML'
        )
        
    return SELECTING_MODE

async def select_mode(update: Update, context: CallbackContext) -> int:
    """Gère la sélection du mode de jeu."""
    query = update.callback_query
    await query.answer()
    
    mode = query.data
    context.user_data['mode'] = mode

    if mode == 'mode_discovery':
        keyboard = InlineKeyboardMarkup([[InlineKeyboardButton("↩️ Назад в главное меню", callback_data='back_to_menu')]])
        await query.edit_message_text(
            text="Вы в режиме 'Справочник'.\nВведите номер, название региона или столицу, чтобы получить информацию.",
            reply_markup=keyboard
        )
        return DISCOVERY_MODE
    else:
        keyboard = get_question_type_keyboard()
        await query.edit_message_text(text="Отлично! Теперь выберите тип вопросов:", reply_markup=keyboard)
        return SELECTING_TYPE

async def select_question_type(update: Update, context: CallbackContext) -> int:
    """Initialise le quiz après la sélection du type de question."""
    query = update.callback_query
    await query.answer()
    
    user_data = context.user_data
    user_data['question_type'] = query.data
    user_data['score'] = 0
    user_data['question_count'] = 0
    user_data['asked_questions'] = []

    mode = user_data['mode']
    if mode == 'mode_challenge':
        user_data['total_questions'] = 10
    elif mode == 'mode_marathon':
        user_data['total_questions'] = len(REGIONS_DATA)
    
    return await send_question(update, context)

async def send_question(update: Update, context: CallbackContext) -> int:
    """Génère et envoie une nouvelle question."""
    query = update.callback_query
    user_data = context.user_data

    # Choisir une question non posée
    available_regions = [r for r in REGIONS_DATA if r['numero'] not in user_data['asked_questions']]
    if not available_regions:
        return await end_quiz(update, context) # Fin si toutes les questions ont été posées
        
    correct_answer_data = random.choice(available_regions)
    user_data['current_question'] = correct_answer_data
    user_data['asked_questions'].append(correct_answer_data['numero'])
    user_data['question_count'] += 1
    
    # Décider du type de question pour cette manche
    question_type = user_data['question_type']
    if question_type == 'type_random':
        user_data['question_type_for_question'] = random.choice(['type_capital', 'type_region', 'type_number'])
    else:
        user_data['question_type_for_question'] = question_type

    # Générer les mauvaises réponses
    options = [correct_answer_data]
    while len(options) < 4:
        wrong_option = random.choice(REGIONS_DATA)
        if wrong_option not in options:
            options.append(wrong_option)
    random.shuffle(options)

    # Créer les boutons
    keyboard_buttons = []
    qt = user_data['question_type_for_question']
    for option in options:
        if qt == 'type_capital':
            text = option['ville']
            callback = 'answer_' + option['numero']
        elif qt == 'type_region':
            text = f"{option['nom']} ({option['numero']})"
            callback = 'answer_' + option['numero']
        else: # type_number
            text = f"{option['nom']} ({option['ville']})"
            callback = 'answer_' + option['numero']
        
        keyboard_buttons.append([InlineKeyboardButton(text, callback_data=callback)])
    
    user_data['correct_answer_callback'] = 'answer_' + correct_answer_data['numero']

    # Ajouter le bouton Quitter
    keyboard_buttons.append([InlineKeyboardButton("🏁 Завершить викторину", callback_data='exit_quiz')])

    # Envoyer le message
    question_text = generate_question_text(context)
    reply_markup = InlineKeyboardMarkup(keyboard_buttons)

    if query:
        await query.edit_message_text(text=question_text, reply_markup=reply_markup, parse_mode='HTML')
    else: # Cas initial
        await update.message.reply_text(text=question_text, reply_markup=reply_markup, parse_mode='HTML')
        
    return IN_QUIZ

# Remplacez votre ancienne fonction handle_answer par celle-ci.
# N'oubliez pas que vous avez déjà "import asyncio" en haut de votre script.

async def handle_answer(update: Update, context: CallbackContext) -> int:
    """
    Gère la réponse de l'utilisateur, met à jour les boutons avec un feedback visuel
    et passe à la question suivante ou termine le quiz.
    """
    query = update.callback_query
    await query.answer()

    user_data = context.user_data
    user_choice_callback = query.data
    correct_answer_callback = user_data['correct_answer_callback']
    is_correct = (user_choice_callback == correct_answer_callback)

    # --- Logique de mise à jour du clavier ---
    original_keyboard = query.message.reply_markup.inline_keyboard
    new_keyboard = []

    for row in original_keyboard:
        button = row[0]
        
        # On ignore le bouton "Quitter"
        if button.callback_data == 'exit_quiz':
            new_keyboard.append([button])
            continue

        new_text = button.text
        
        if button.callback_data == correct_answer_callback:
            # Si c'est la bonne réponse, on ajoute toujours le check vert
            new_text = f"✅ {button.text}"
        elif button.callback_data == user_choice_callback:
            # Si c'est le choix de l'utilisateur ET qu'il est incorrect
            new_text = f"❌ {button.text}"
        
        # On crée un nouveau bouton avec le texte modifié et on le désactive
        # en lui donnant un callback_data qui ne fait rien ('noop' = no operation)
        new_keyboard.append([InlineKeyboardButton(new_text, callback_data='noop')])

    # On met à jour le message avec le nouveau clavier "corrigé"
    # On garde le texte de la question originale pour le contexte
    question_text = generate_question_text(context)
    await query.edit_message_text(
        text=question_text, 
        reply_markup=InlineKeyboardMarkup(new_keyboard),
        parse_mode='HTML'
    )

    # --- Logique de progression du quiz ---
    if is_correct:
        user_data['score'] = user_data.get('score', 0) + 1
    else:
        # Si le mode est 'Survie', une mauvaise réponse termine le jeu
        if user_data['mode'] == 'mode_survival':
            score = user_data.get('score', 0)
            best_score = context.bot_data.get(BEST_SCORE_KEY, 0)
            if score > best_score:
                context.bot_data[BEST_SCORE_KEY] = score
                best_score = score
            
            # On attend 2 secondes pour que l'utilisateur voie la correction
            await asyncio.sleep(2)

            text = (
                f"☠️ <b>ИГРА ОКОНЧЕНА</b> ☠️\n\n"
                f"Ваш итоговый счёт в режиме 'Выживание': {score}\n"
                f"Лучший счёт: {best_score}\n\n"
                "Хотите сыграть снова?"
            )
            keyboard = InlineKeyboardMarkup([[InlineKeyboardButton("Сыграть снова", callback_data='play_again')]])
            await query.edit_message_text(text=text, reply_markup=keyboard, parse_mode='HTML')
            return SELECTING_MODE

    # Pause de 2 secondes pour que l'utilisateur voie le résultat
    await asyncio.sleep(2) 
    
    # Vérifier la fin du quiz (challenge/marathon)
    if user_data['mode'] in ['mode_challenge', 'mode_marathon']:
        if user_data['question_count'] >= user_data['total_questions']:
            return await end_quiz(update, context)

    # Passer à la question suivante
    return await send_question(update, context)



async def end_quiz(update: Update, context: CallbackContext) -> int:
    """Termine le quiz et affiche le score final."""
    query = update.callback_query
    score = context.user_data.get('score', 0)
    total = context.user_data.get('total_questions', context.user_data.get('question_count', 0))
    
    text = f"🎉 <b>Викторина завершена!</b> 🎉\n\nВаш итоговый счёт: {score} из {total}"
    
    keyboard = InlineKeyboardMarkup([[InlineKeyboardButton("↩️ Назад в главное меню", callback_data='back_to_menu')]])
    
    if query:
        await query.answer()
        await query.edit_message_text(text=text, reply_markup=keyboard, parse_mode='HTML')
    
    return SELECTING_MODE

async def exit_quiz(update: Update, context: CallbackContext) -> int:
    """Permet à l'utilisateur de quitter le quiz en cours."""
    await end_quiz(update, context)
    return SELECTING_MODE

async def discovery_search(update: Update, context: CallbackContext) -> int:
    """Recherche une région et affiche ses informations."""
    user_text = update.message.text.lower().strip()
    
    results = [r for r in REGIONS_DATA if user_text in r['nom'].lower() or user_text in r['ville'].lower() or user_text == r['numero']]
    
    if results:
        message = "🔎 <b>Результаты поиска:</b>\n\n"
        for res in results:
            message += f"• <b>{res['nom']} ({res['numero']})</b>\n  Столица: {res['ville']}\n\n"
    else:
        message = "😕 По вашему запросу ничего не найдено."

    keyboard = InlineKeyboardMarkup([[InlineKeyboardButton("↩️ Назад в главное меню", callback_data='back_to_menu')]])
    # <-- MODIFIÉ : parse_mode='HTML'
    await update.message.reply_text(message, reply_markup=keyboard, parse_mode='HTML')
    
    return DISCOVERY_MODE

def main() -> None:
    """Démarre le bot."""
    application = Application.builder().token(TELEGRAM_API_KEY).build()

    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            SELECTING_MODE: [
                CallbackQueryHandler(select_mode, pattern='^mode_'),
                CallbackQueryHandler(start, pattern='^play_again$'),
		CallbackQueryHandler(start, pattern='^back_to_menu$')
            ],
            SELECTING_TYPE: [
                CallbackQueryHandler(select_question_type, pattern='^type_'),
                CallbackQueryHandler(start, pattern='^back_to_menu$')
            ],
            IN_QUIZ: [
                CallbackQueryHandler(handle_answer, pattern='^answer_'),
                CallbackQueryHandler(exit_quiz, pattern='^exit_quiz$')
            ],
            DISCOVERY_MODE: [
                MessageHandler(filters.TEXT & ~filters.COMMAND, discovery_search),
                CallbackQueryHandler(start, pattern='^back_to_menu$')
            ],
        },
        fallbacks=[CommandHandler('start', start)],
    )

    application.add_handler(conv_handler)

    logger.info("Бот запускается...")
    application.run_polling()

if __name__ == '__main__':
    main()
