import requests
from bs4 import BeautifulSoup
import smtplib
from email.message import EmailMessage
import schedule
import time
from datetime import datetime

# Configuration
GMAIL_USER = "vanshikakanwar243@gmail.com"
GMAIL_APP_PASSWORD = "unwj tacr orhk icgc"
RECIPIENT_EMAIL = "vanshikakanwar243@gmail.com"
BDG_URL = "https://bdgwinvip9.com/#/saasLottery/K3?gameCode=K3_1M&lottery=K3"

# Tracking variables
predictions_today = []
daily_stats = {'total': 0, 'correct': 0, 'wrong': 0, 'profit': 0}
MAX_DAILY_PREDICTIONS = 10

def send_email(subject, body):
    try:
        msg = EmailMessage()
        msg['Subject'] = subject
        msg['From'] = GMAIL_USER
        msg['To'] = RECIPIENT_EMAIL
        msg.set_content(body)

        with smtplib.SMTP_SSL('smtp.gmail.com', 465) as smtp:
            smtp.login(GMAIL_USER, GMAIL_APP_PASSWORD)
            smtp.send_message(msg)
        print("Email sent successfully")
    except Exception as e:
        print(f"Email error: {str(e)}")

def scrape_results():
    try:
        headers = {'User-Agent': 'Mozilla/5.0'}
        response = requests.get(BDG_URL, headers=headers, timeout=10)
        soup = BeautifulSoup(response.text, 'html.parser')
        
        # Update these selectors based on BDG site structure
        result = int(soup.select_one('.result-class').text.strip())
        period = soup.select_one('.period-class').text.strip()
        return {'number': result, 'period': period}
    except Exception as e:
        print(f"Scraping error: {str(e)}")
        return None

def analyze_pattern(history):
    """70% algorithm + 30% strategy logic"""
    if len(history) < 5:
        return None, 0
    
    # Sample prediction logic (customize this)
    last_results = [h['number'] for h in history[-5:]]
    
    # If 3 appeared twice in last 5 rounds
    if last_results.count(3) >= 2:
        return 18, 75  # Predict 18 with 75% confidence
    
    # If 18 appeared twice in last 5 rounds
    elif last_results.count(18) >= 2:
        return 3, 80   # Predict 3 with 80% confidence
    
    return None, 0

def make_prediction():
    global predictions_today, daily_stats
    
    if daily_stats['total'] >= MAX_DAILY_PREDICTIONS:
        return
    
    # Get latest results
    data = scrape_results()
    if not data:
        return
    
    # Analyze pattern
    prediction, confidence = analyze_pattern(predictions_today)
    
    # Only proceed if high confidence
    if prediction in [3, 18] and confidence >= 70:
        alert_msg = f"3 ya 18 aa sakta hai\nPeriod: {data['period']}\nTime: {datetime.now().strftime('%I:%M %p')}\nConfidence: {confidence}%"
        send_email("K3 Prediction Alert", alert_msg)
        
        predictions_today.append({
            'number': prediction,
            'period': data['period'],
            'time': datetime.now(),
            'confidence': confidence,
            'verified': False
        })
        daily_stats['total'] += 1

def verify_predictions():
    global daily_stats
    
    current_data = scrape_results()
    if not current_data:
        return
    
    for pred in predictions_today:
        if not pred.get('verified') and pred['period'] == current_data['period']:
            is_correct = pred['number'] == current_data['number']
            pred['verified'] = True
            pred['correct'] = is_correct
            
            if is_correct:
                daily_stats['correct'] += 1
                daily_stats['profit'] += 60
            else:
                daily_stats['wrong'] += 1
                daily_stats['profit'] -= 60

def send_daily_summary():
    subject = f"K3 Daily Summary - {datetime.now().date()}"
    
    if predictions_today:
        accuracy = (daily_stats['correct'] / daily_stats['total']) * 100
        body = f"""📊 Predictions: {daily_stats['total']}
✅ Correct: {daily_stats['correct']}
❌ Wrong: {daily_stats['wrong']}
🎯 Accuracy: {accuracy:.2f}%
💰 Profit/Loss: ₹{daily_stats['profit']}

System Status: ACTIVE"""
    else:
        body = """🔍 No predictions today
Reason: Unstable patterns/low confidence
📊 Trades: 0
🎯 Accuracy: N/A
💰 Profit/Loss: N/A

System Status: ACTIVE (Monitoring)"""

    send_email(subject, body)
    # Reset for next day
    predictions_today.clear()
    daily_stats = {'total': 0, 'correct': 0, 'wrong': 0, 'profit': 0}

# Scheduling
schedule.every(1).minutes.do(make_prediction)
schedule.every(1).minutes.do(verify_predictions)
schedule.every().day.at("22:00").do(send_daily_summary)  # 10 PM IST

if __name__ == "__main__":
    print("K3 Prediction System Started")
    while True:
        schedule.run_pending()
        time.sleep(1)
