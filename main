from flask import Flask, render_template, request, redirect, url_for, jsonify
from flask_sqlalchemy import SQLAlchemy
from apscheduler.schedulers.background import BackgroundScheduler
import feedparser
import datetime
import os

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///data.db'
db = SQLAlchemy(app)

# Models
class Article(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(300))
    summary = db.Column(db.Text)
    url = db.Column(db.String(500))
    source = db.Column(db.String(100))
    date_published = db.Column(db.DateTime, default=datetime.datetime.utcnow)

# Scraper logic (using RSS feeds for now)
FEEDS = {
    'PubMed': 'https://pubmed.ncbi.nlm.nih.gov/rss/search/1X2oNUnWsZgAe0Db1U7cJ6sBDnV6bZbiQDQ_RV6nJhQ6LUJ5QJ/?limit=10&utm_campaign=pubmed-2&fc=20240130131332',
    'Frontiers in Psychology': 'https://www.frontiersin.org/journals/psychology/rss',
    'Nature Neuroscience': 'https://www.nature.com/subjects/neuroscience/rss'
}

def fetch_articles():
    for source, url in FEEDS.items():
        feed = feedparser.parse(url)
        for entry in feed.entries:
            exists = Article.query.filter_by(url=entry.link).first()
            if not exists:
                article = Article(
                    title=entry.title,
                    summary=entry.summary,
                    url=entry.link,
                    source=source
                )
                db.session.add(article)
    db.session.commit()

# Scheduler to run scraper every 24h
scheduler = BackgroundScheduler()
scheduler.add_job(fetch_articles, 'interval', hours=24)
scheduler.start()

# Routes
@app.route('/')
def index():
    q = request.args.get('q')
    if q:
        articles = Article.query.filter(Article.title.ilike(f'%{q}%')).order_by(Article.date_published.desc()).all()
    else:
        articles = Article.query.order_by(Article.date_published.desc()).all()
    return render_template('index.html', articles=articles)

@app.route('/article/<int:article_id>')
def article_detail(article_id):
    article = Article.query.get_or_404(article_id)
    return render_template('article.html', article=article)

@app.route('/about')
def about():
    return render_template('about.html')

# Run setup if needed
if not os.path.exists('data.db'):
    db.create_all()
    fetch_articles()

if __name__ == '__main__':
    app.run(debug=True)
