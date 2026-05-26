---
applyTo: "**/*.py"
description: "Flask and Python web development best practices. Use when: working with Flask routes, blueprints, SQLAlchemy models, API endpoints, Jinja templates"
---

# Flask Development Guidelines

## MCAPS Security (Mandatory)

- **NEVER** hardcode credentials, API keys, or connection strings with passwords
- Use `DefaultAzureCredential` for all Azure service authentication
- Load configuration from environment variables only
- Use Azure Key Vault references for App Service settings

```python
# CORRECT - Azure authentication
from azure.identity import DefaultAzureCredential
credential = DefaultAzureCredential()

# CORRECT - Configuration from environment
import os
DATABASE_URL = os.environ.get('DATABASE_URL')

# WRONG - Never do this
API_KEY = "sk-abc123..."  # ❌ Hardcoded secret
```

## Flask Patterns

### App Factory (Required)
```python
def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)
    
    # Initialize extensions
    db.init_app(app)
    migrate.init_app(app, db)
    
    # Register blueprints
    from app.api import bp as api_bp
    app.register_blueprint(api_bp, url_prefix='/api')
    
    return app
```

### Blueprints for Organization
```python
from flask import Blueprint

bp = Blueprint('api', __name__)

@bp.route('/health')
def health_check():
    return {'status': 'healthy'}
```

### Error Handling
```python
@app.errorhandler(404)
def not_found(error):
    return {'error': 'Not found'}, 404

@app.errorhandler(500)
def internal_error(error):
    db.session.rollback()
    return {'error': 'Internal server error'}, 500
```

## SQLAlchemy Models

```python
from datetime import datetime
from app import db

class User(db.Model):
    __tablename__ = 'users'
    
    id: int = db.Column(db.Integer, primary_key=True)
    email: str = db.Column(db.String(120), unique=True, nullable=False, index=True)
    created_at: datetime = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relationships
    posts = db.relationship('Post', backref='author', lazy='dynamic')
    
    def to_dict(self) -> dict:
        return {
            'id': self.id,
            'email': self.email,
            'created_at': self.created_at.isoformat()
        }
```

## Type Hints Required

Always include type hints:
```python
from typing import Optional

def get_user_by_email(email: str) -> Optional[User]:
    return User.query.filter_by(email=email).first()
```
