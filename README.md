import vk_api
from vk_api.bot_longpoll import VkBotLongPoll, VkBotEventType
from vk_api.utils import get_random_id

# Настройки
GROUP_ID = 'ID_ВАШЕЙ_ГРУППЫ'  # ID вашего сообщества
TOKEN = 'ВАШ_ТОКЕН'  # Токен доступа к API

# Инициализация VK API
vk_session = vk_api.VkApi(token=TOKEN)
vk = vk_session.get_api()
longpoll = VkBotLongPoll(vk_session, GROUP_ID)

# Список для хранения ID пользователей, которым уже отправлено приветствие
greeted_users = set()

def send_message(user_id, message=None, attachment=None):
    """Отправка сообщения пользователю."""
    vk.messages.send(
        user_id=user_id,
        message=message,
        random_id=get_random_id(),
        attachment=attachment
    )

def handle_event(event):
    """Обработка событий от Long Poll API."""
    if event.type == VkBotEventType.MESSAGE_NEW:
        user_id = event.obj.message['from_id']
        text = event.obj.message.get('text', '').strip()
        attachments = event.obj.message.get('attachments', [])

        # Проверка, является ли это первое сообщение пользователя
        if user_id not in greeted_users:
            send_message(user_id, "Привет! Я бот. Отправь мне изображение, и я отправлю его обратно!")
            greeted_users.add(user_id)

        # Обработка изображений
        if attachments:
            for attachment in attachments:
                if attachment['type'] == 'photo':
                    # Получаем URL изображения
                    photo = attachment['photo']
                    photo_url = None
                    for size in photo['sizes']:
                        if size['type'] == 'z':  # Выбираем размер 'z' (оптимальный)
                            photo_url = size['url']
                            break

                    if photo_url:
                        # Отправляем изображение обратно
                        send_message(user_id, attachment=photo_url)

# Основной цикл бота
def main():
    print("Бот запущен...")
    for event in longpoll.listen():
        handle_event(event)

if __name__ == '__main__':
    main()

