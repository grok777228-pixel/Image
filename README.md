import asyncio
import logging
import random
import string
import urllib.parse
from aiogram import Bot, Dispatcher, types, F
from aiogram.filters.command import Command

# Настройки
API_TOKEN = '8598978154:AAEYTaQJoMVZoomSz3tFJ7KdcqU3wHtJRgM'
POLLINATIONS_BASE_URL = "https://pollinations.ai/p/"

# Логирование
logging.basicConfig(level=logging.INFO)

# Инициализация бота и диспетчера
bot = Bot(token=API_TOKEN)
dp = Dispatcher()

@dp.message(Command("start"))
async def cmd_start(message: types.Message):
    await message.answer(
        "👋 Привет! Я бот для генерации изображений.\n\n"
        "Просто напиши мне, что ты хочешь увидеть (например: 'космонавт на марсе' или 'cyberpunk city'), и я создам это для тебя совершенно бесплатно!"
    )

@dp.message(F.text)
async def generate_image(message: types.Message):
    prompt = message.text
    # Отправляем уведомление о начале работы
    wait_message = await message.answer(f"🎨 Генерирую '{prompt}'... Подождите немного.")
    
    try:
        # Кодируем запрос для URL
        encoded_prompt = urllib.parse.quote(prompt)
        
        # Добавляем случайное число, чтобы избежать кэширования одинаковых запросов
        seed = random.randint(1, 1000000)
        
        # Формируем прямую ссылку на изображение от Pollinations.ai
        image_url = f"{POLLINATIONS_BASE_URL}{encoded_prompt}?width=1024&height=1024&seed={seed}&model=flux"
        
        # Отправляем фото пользователю напрямую по URL
        await message.answer_photo(
            photo=image_url,
            caption=f"✅ Готово по запросу: {prompt}\n\nСгенерировано с помощью Pollinations.ai"
        )
        
        # Удаляем сообщение о ожидании
        await wait_message.delete()
        
    except Exception as e:
        logging.error(f"Ошибка генерации: {e}")
        await wait_message.edit_text("❌ Произошла ошибка при генерации. Попробуйте другой запрос.")

async def main():
    print("Бот запущен и готов к работе!")
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
