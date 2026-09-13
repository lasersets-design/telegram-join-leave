import os
from telegram import Update
from telegram.ext import Application, ChatMemberHandler, ContextTypes

BOT_TOKEN = os.environ["BOT_TOKEN"]
OWNER_ID = int(os.environ["OWNER_ID"])


async def member_status_changed(
    update: Update,
    context: ContextTypes.DEFAULT_TYPE
):
    change = update.chat_member

    old_status = change.old_chat_member.status
    new_status = change.new_chat_member.status
    user = change.new_chat_member.user

    # عضو جدید وارد گروه شده
    joined = (
        old_status in ["left", "kicked"]
        and new_status in ["member", "administrator", "creator"]
    )

    # عضو گروه را ترک کرده
    left = (
        old_status in ["member", "administrator"]
        and new_status in ["left", "kicked"]
    )

    if joined:
        text = (
            "🟢 JOIN\n\n"
            f"👤 {user.full_name}\n"
            f"🆔 {user.id}\n"
            f"🔗 @{user.username}" if user.username else
            f"🟢 JOIN\n\n👤 {user.full_name}\n🆔 {user.id}"
        )

        await context.bot.send_message(
            chat_id=OWNER_ID,
            text=text
        )

    elif left:
        text = (
            "🔴 LEAVE\n\n"
            f"👤 {user.full_name}\n"
            f"🆔 {user.id}\n"
            f"🔗 @{user.username}" if user.username else
            f"🔴 LEAVE\n\n👤 {user.full_name}\n🆔 {user.id}"
        )

        await context.bot.send_message(
            chat_id=OWNER_ID,
            text=text
        )


def main():
    app = Application.builder().token(BOT_TOKEN).build()

    app.add_handler(
        ChatMemberHandler(
            member_status_changed,
            ChatMemberHandler.CHAT_MEMBER
        )
    )

    app.run_polling(
        allowed_updates=["chat_member"]
    )


if __name__ == "__main__":
    main()
