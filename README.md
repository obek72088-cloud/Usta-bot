# USTA BOT — MVP

Telegram bot where masters register themselves for free and customers search by service and location.

## 1. Create a bot
Open Telegram and use @BotFather.
Run `/newbot`, choose a name and username, then copy the token.

## 2. Install
Python 3.11+ recommended.

```bash
pip install -r requirements.txt
```

## 3. Set token

Linux/macOS:
```bash
export BOT_TOKEN="YOUR_TOKEN"
python bot.py
```

Windows PowerShell:
```powershell
$env:BOT_TOKEN="YOUR_TOKEN"
python bot.py
```

## 4. MVP features
- Free master registration
- Service/category
- City/district
- Experience
- Description
- Work photo
- Customer search
- Direct phone display
- SQLite database
- Basic master profile

## Next version
- admin moderation
- verification badge
- ratings/reviews
- order system
- PRO placement
- analytics
- anti-spam
- payments
- web/mobile frontend
