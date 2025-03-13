from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, filters, CallbackContext, ConversationHandler
import os

# Bot Token
TOKEN = "7811287079:AAH9NzGy6jVgMpBEAGXvR9-bxAc22vcuflw"
ADMIN_CHAT_ID = "@BrouwnT"  # Telegram username where patient reports will be sent

# Define states for the conversation
(
    GENERAL_INFO, PAIN_LOCATION, PAIN_TYPE, SEVERITY, PAIN_TRIGGER, RELIEF, MOBILITY, WEAKNESS, NEUROLOGICAL, INJURY_HISTORY, SCAPULAR_ASSESSMENT, DIAGNOSIS
) = range(12)
user_data = {}

def start(update: Update, context: CallbackContext):
    update.message.reply_text("Hello! Welcome to PT_Ziyad_Bot, your comprehensive shoulder & scapular pain self-assessment assistant.\n\nLet's begin by gathering some general information.\n\nWhat is your age?")
    return GENERAL_INFO

def general_info(update: Update, context: CallbackContext):
    user_data['age'] = update.message.text
    update.message.reply_text("What is your gender? (Male/Female/Other)")
    return PAIN_LOCATION

def pain_location(update: Update, context: CallbackContext):
    user_data['gender'] = update.message.text
    keyboard = [
        [InlineKeyboardButton("Front of Shoulder", callback_data='front')],
        [InlineKeyboardButton("Side of Shoulder", callback_data='side')],
        [InlineKeyboardButton("Back of Shoulder", callback_data='back')],
        [InlineKeyboardButton("Shoulder Blade", callback_data='scapula')],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    update.message.reply_text("Where is your pain located?", reply_markup=reply_markup)
    return PAIN_TYPE

def pain_type(update: Update, context: CallbackContext):
    query = update.callback_query
    query.answer()
    user_data['pain_location'] = query.data
    keyboard = [
        [InlineKeyboardButton("Sharp/Stabbing", callback_data='sharp')],
        [InlineKeyboardButton("Dull/Aching", callback_data='dull')],
        [InlineKeyboardButton("Burning", callback_data='burning')],
        [InlineKeyboardButton("Tingling/Numbness", callback_data='tingling')],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    query.edit_message_text("How would you describe your pain?", reply_markup=reply_markup)
    return SEVERITY

def severity(update: Update, context: CallbackContext):
    query = update.callback_query
    query.answer()
    user_data['pain_type'] = query.data
    keyboard = [
        [InlineKeyboardButton("Mild (1-3)", callback_data='mild')],
        [InlineKeyboardButton("Moderate (4-6)", callback_data='moderate')],
        [InlineKeyboardButton("Severe (7-9)", callback_data='severe')],
        [InlineKeyboardButton("Unbearable (10)", callback_data='unbearable')],
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    query.edit_message_text("How severe is your pain (0-10)?", reply_markup=reply_markup)
    return PAIN_TRIGGER

def diagnosis(update: Update, context: CallbackContext):
    query = update.callback_query
    query.answer()
    user_data['pain_severity'] = query.data
    
    # Diagnosis Logic
    if user_data['pain_type'] == 'sharp' and user_data['pain_severity'] == 'moderate':
        diagnosis_text = "Possible Rotator Cuff Injury. Pain with overhead movement suggests inflammation or tendon damage."
    elif user_data['pain_type'] == 'dull' and user_data['pain_severity'] == 'severe':
        diagnosis_text = "Possible Frozen Shoulder. Severe stiffness and limited movement indicate adhesive capsulitis."
    elif user_data['pain_type'] == 'burning' and user_data['pain_location'] == 'scapula':
        diagnosis_text = "Possible Nerve Compression or Thoracic Outlet Syndrome. Burning pain in the scapula suggests nerve involvement."
    else:
        diagnosis_text = "Further assessment by a physiotherapist is recommended to pinpoint the exact cause."
    
    summary = (f"Assessment Summary for {user_data.get('age', 'Unknown Age')} (Gender: {user_data.get('gender', 'Unknown')}):\n"
               f"- Pain Location: {user_data.get('pain_location', 'Not specified')}\n"
               f"- Pain Type: {user_data.get('pain_type', 'Not specified')}\n"
               f"- Pain Severity: {user_data.get('pain_severity', 'Not specified')}\n"
               f"Diagnosis: {diagnosis_text}")
    
    query.edit_message_text(summary)
    
    # Send report to the admin
    context.bot.send_message(chat_id=ADMIN_CHAT_ID, text=f"New Patient Report\n\n{summary}")
    
    return ConversationHandler.END

def main():
    app = Application.builder().token(TOKEN).build()
    
    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            GENERAL_INFO: [MessageHandler(filters.TEXT & ~filters.COMMAND, general_info)],
            PAIN_LOCATION: [MessageHandler(filters.TEXT & ~filters.COMMAND, pain_location)],
            PAIN_TYPE: [MessageHandler(filters.TEXT & ~filters.COMMAND, pain_type)],
            SEVERITY: [MessageHandler(filters.TEXT & ~filters.COMMAND, severity)],
            DIAGNOSIS: [MessageHandler(filters.TEXT & ~filters.COMMAND, diagnosis)],
        },
        fallbacks=[CommandHandler('cancel', lambda update, context: update.message.reply_text("Assessment canceled."))],
    )
    
    app.add_handler(conv_handler)
    app.run_polling()

if __name__ == "__main__":
    main()
