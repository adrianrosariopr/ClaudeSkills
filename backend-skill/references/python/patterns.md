<overview>
Python backend patterns covering Django, FastAPI, SQLAlchemy, async Python, and Pydantic. Modern practices for building robust Python APIs.
</overview>

<project_structure>
**Django Structure:**

```
project/
├── config/              # Project settings
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── users/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   └── tests.py
│   └── posts/
├── core/                # Shared utilities
├── manage.py
└── requirements.txt
```

**FastAPI Structure:**

```
app/
├── api/
│   ├── v1/
│   │   ├── endpoints/
│   │   │   ├── users.py
│   │   │   └── posts.py
│   │   └── router.py
│   └── deps.py          # Dependencies
├── core/
│   ├── config.py
│   └── security.py
├── models/              # SQLAlchemy models
├── schemas/             # Pydantic schemas
├── services/            # Business logic
├── db/
│   └── session.py
└── main.py
```
</project_structure>

<django_patterns>
**Django REST Framework:**

```python
# models.py
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    email = models.EmailField(unique=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

class Post(models.Model):
    title = models.CharField(max_length=255)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    published = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-created_at']
        indexes = [models.Index(fields=['author', 'published'])]

# serializers.py
from rest_framework import serializers

class PostSerializer(serializers.ModelSerializer):
    author = serializers.StringRelatedField(read_only=True)

    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author', 'published', 'created_at']
        read_only_fields = ['author', 'created_at']

# views.py
from rest_framework import viewsets, permissions
from rest_framework.decorators import action
from rest_framework.response import Response

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.select_related('author')
    serializer_class = PostSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def get_queryset(self):
        queryset = super().get_queryset()
        if self.action == 'list':
            queryset = queryset.filter(published=True)
        return queryset

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        post = self.get_object()
        post.published = True
        post.save()
        return Response({'status': 'published'})
```

**Query Optimization:**

```python
# Prevent N+1 with select_related (FK) and prefetch_related (M2M)
posts = Post.objects.select_related('author').prefetch_related('tags').all()

# Only fetch needed fields
users = User.objects.only('id', 'email', 'name').all()

# Aggregate queries
from django.db.models import Count, Avg

stats = Post.objects.aggregate(
    total=Count('id'),
    avg_views=Avg('view_count')
)

# Annotate for computed fields
authors = User.objects.annotate(post_count=Count('posts'))
```
</django_patterns>

<fastapi_patterns>
**FastAPI Setup:**

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="My API", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
from app.api.v1 import router as api_router
app.include_router(api_router, prefix="/api/v1")
```

**Endpoints with Pydantic:**

```python
from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel, EmailStr
from typing import List, Optional

router = APIRouter(prefix="/users", tags=["users"])

# Pydantic schemas
class UserCreate(BaseModel):
    email: EmailStr
    password: str
    name: Optional[str] = None

class UserResponse(BaseModel):
    id: int
    email: EmailStr
    name: Optional[str]

    class Config:
        from_attributes = True  # For ORM mode

class UserList(BaseModel):
    data: List[UserResponse]
    total: int

# Endpoints
@router.post("/", response_model=UserResponse, status_code=201)
async def create_user(
    user_data: UserCreate,
    db: Session = Depends(get_db),
):
    if await user_service.get_by_email(db, user_data.email):
        raise HTTPException(400, "Email already registered")

    user = await user_service.create(db, user_data)
    return user

@router.get("/", response_model=UserList)
async def list_users(
    skip: int = 0,
    limit: int = 20,
    db: Session = Depends(get_db),
):
    users = await user_service.get_all(db, skip=skip, limit=limit)
    total = await user_service.count(db)
    return {"data": users, "total": total}

@router.get("/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    db: Session = Depends(get_db),
):
    user = await user_service.get(db, user_id)
    if not user:
        raise HTTPException(404, "User not found")
    return user
```

**Dependencies:**

```python
from fastapi import Depends, HTTPException, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Security(security),
    db: Session = Depends(get_db),
) -> User:
    token = credentials.credentials
    payload = decode_jwt(token)
    if not payload:
        raise HTTPException(401, "Invalid token")

    user = await user_service.get(db, payload["sub"])
    if not user:
        raise HTTPException(401, "User not found")
    return user

# Use in endpoint
@router.get("/me")
async def get_me(user: User = Depends(get_current_user)):
    return user
```
</fastapi_patterns>

<sqlalchemy>
**SQLAlchemy Models:**

```python
from sqlalchemy import Column, Integer, String, ForeignKey, Boolean, DateTime
from sqlalchemy.orm import relationship, declarative_base
from sqlalchemy.sql import func

Base = declarative_base()

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    name = Column(String(255))
    created_at = Column(DateTime, server_default=func.now())

    posts = relationship("Post", back_populates="author")

class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True)
    title = Column(String(255), nullable=False)
    content = Column(String)
    published = Column(Boolean, default=False)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False, index=True)
    created_at = Column(DateTime, server_default=func.now())

    author = relationship("User", back_populates="posts")
```

**Async SQLAlchemy (2.0):**

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
async_session = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_db():
    async with async_session() as session:
        yield session

# Queries
from sqlalchemy import select
from sqlalchemy.orm import selectinload

async def get_posts_with_authors(db: AsyncSession):
    result = await db.execute(
        select(Post)
        .options(selectinload(Post.author))
        .where(Post.published == True)
        .order_by(Post.created_at.desc())
    )
    return result.scalars().all()
```
</sqlalchemy>

<async_patterns>
**Async Python:**

```python
import asyncio
from typing import List

# Concurrent requests
async def fetch_all_data():
    async with aiohttp.ClientSession() as session:
        tasks = [
            fetch_users(session),
            fetch_posts(session),
            fetch_comments(session),
        ]
        users, posts, comments = await asyncio.gather(*tasks)
        return {"users": users, "posts": posts, "comments": comments}

# Background tasks in FastAPI
from fastapi import BackgroundTasks

@router.post("/orders")
async def create_order(
    order: OrderCreate,
    background_tasks: BackgroundTasks,
    db: Session = Depends(get_db),
):
    order = await order_service.create(db, order)
    background_tasks.add_task(send_confirmation_email, order.user.email, order.id)
    return order

# Async context manager
from contextlib import asynccontextmanager

@asynccontextmanager
async def get_redis():
    redis = await aioredis.from_url("redis://localhost")
    try:
        yield redis
    finally:
        await redis.close()
```
</async_patterns>

<celery>
**Celery Tasks:**

```python
from celery import Celery

celery = Celery('tasks', broker='redis://localhost:6379/0')

@celery.task(bind=True, max_retries=3)
def send_email(self, user_id: int, template: str):
    try:
        user = User.objects.get(id=user_id)
        # Send email
    except Exception as exc:
        self.retry(exc=exc, countdown=60 * (2 ** self.request.retries))

@celery.task
def process_order(order_id: int):
    order = Order.objects.get(id=order_id)
    # Process

# Dispatch
send_email.delay(user.id, 'welcome')
send_email.apply_async(args=[user.id, 'welcome'], countdown=60)

# Periodic tasks
from celery.schedules import crontab

celery.conf.beat_schedule = {
    'cleanup-daily': {
        'task': 'tasks.cleanup_old_data',
        'schedule': crontab(hour=0, minute=0),
    },
}
```
</celery>

<testing>
**Testing with pytest:**

```python
import pytest
from fastapi.testclient import TestClient
from app.main import app
from app.db import get_db

@pytest.fixture
def client(db_session):
    def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db
    with TestClient(app) as client:
        yield client
    app.dependency_overrides.clear()

def test_create_user(client):
    response = client.post("/api/v1/users", json={
        "email": "test@example.com",
        "password": "password123",
    })

    assert response.status_code == 201
    assert response.json()["email"] == "test@example.com"

def test_get_user_not_found(client):
    response = client.get("/api/v1/users/999")
    assert response.status_code == 404

# Async tests
import pytest_asyncio

@pytest_asyncio.fixture
async def async_client():
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.mark.asyncio
async def test_async_endpoint(async_client):
    response = await async_client.get("/api/v1/users")
    assert response.status_code == 200
```
</testing>

<quick_reference>
**Common Commands:**

```bash
# Django
python manage.py runserver
python manage.py migrate
python manage.py createsuperuser
python manage.py test

# FastAPI
uvicorn app.main:app --reload
alembic upgrade head
alembic revision --autogenerate -m "description"

# Celery
celery -A tasks worker --loglevel=info
celery -A tasks beat

# Testing
pytest
pytest -v --cov=app
```
</quick_reference>
