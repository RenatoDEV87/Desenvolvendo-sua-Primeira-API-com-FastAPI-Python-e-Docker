# Desenvolvendo-sua-Primeira-API-com-FastAPI-Python-e-Docker
 Desenvolvendo sua Primeira API com FastAPI, Python e Docker

 # WorkoutAPI

WorkoutAPI é uma API de competição de crossfit desenvolvida utilizando o framework FastAPI. Este projeto visa fornecer um exemplo prático e simplificado de como utilizar FastAPI junto com outras bibliotecas populares.

## Características Principais do FastAPI
- **Alta Performance**: O FastAPI é um dos frameworks mais rápidos disponíveis para a construção de APIs em Python.
- **Fácil de Aprender e Programar**: A simplicidade e a clareza do código permitem que desenvolvedores aprendam e implementem funcionalidades rapidamente.
- **Pronto para Produção**: Possui suporte integrado para documentação automática e validação de dados.

## Pilha da API
- **FastAPI (assíncrono)**: Para a construção da API.
- **Alembic e SQLAlchemy**: Para migrações e manipulação de banco de dados.
- **Pydantic**: Para a validação de dados.
- **PostgreSQL via Docker**: Para armazenamento de dados.

## Configuração do Ambiente de Desenvolvimento

### 1. Configuração do Ambiente Virtual
```bash
pyenv virtualenv 3.11.4 workoutapi
pyenv activate workoutapi
pip install -r requirements.txt
```
### 2. Subir o Banco de Dados
```bash
make run-docker
```
### 3. Criar Migrações
```bash
make create-migrations d="nome_da_migration"
```
### 4. Executar Migrações
```bash
make run-migrations
```
### 5. Subir a API
```bash
make run
```
Acesse: http://127.0.0.1:8000/docs
### Desafio Final
1. Adicionar Query Parameters nos Endpoints
```bash
from fastapi import FastAPI, Query
from typing import List, Optional
from sqlalchemy.orm import Session
from database import get_db
from models import Atleta
from schemas import AtletaResponse

app = FastAPI()

@app.get("/atletas/", response_model=List[AtletaResponse])
def get_atletas(nome: Optional[str] = Query(None), cpf: Optional[str] = Query(None), db: Session = Depends(get_db)):
    query = db.query(Atleta)
    if nome:
        query = query.filter(Atleta.nome == nome)
    if cpf:
        query = query.filter(Atleta.cpf == cpf)
    return query.all()
```
### 2. Customizar Response de Retorno dos Endpoints
```bash
from pydantic import BaseModel
from typing import List

class AtletaCustomResponse(BaseModel):
    nome: str
    centro_treinamento: str
    categoria: str

@app.get("/atletas/", response_model=List[AtletaCustomResponse])
def get_all_atletas(db: Session = Depends(get_db)):
    atletas = db.query(Atleta).all()
    return [{"nome": atleta.nome, "centro_treinamento": atleta.centro_treinamento, "categoria": atleta.categoria} for atleta in atletas]
```
### 3. Manipular Exceção de Integridade dos Dados
```bash
from fastapi import HTTPException
from sqlalchemy.exc import IntegrityError

@app.post("/atletas/")
def create_atleta(atleta: AtletaCreate, db: Session = Depends(get_db)):
    new_atleta = Atleta(**atleta.dict())
    db.add(new_atleta)
    try:
        db.commit()
    except IntegrityError:
        db.rollback()
        raise HTTPException(status_code=303, detail=f"Já existe um atleta cadastrado com o cpf: {atleta.cpf}")
    return new_atleta
```
### 4. Adicionar Paginação utilizando a biblioteca fastapi-pagination
```bash
from fastapi_pagination import Page, add_pagination, paginate

@app.get("/atletas/", response_model=Page[AtletaCustomResponse])
def get_all_atletas_paginated(db: Session = Depends(get_db)):
    atletas = db.query(Atleta).all()
    custom_response = [{"nome": atleta.nome, "centro_treinamento": atleta.centro_treinamento, "categoria": atleta.categoria} for atleta in atletas]
    return paginate(custom_response)

add_pagination(app)
```
### Estrutura Final dos Arquivos
models.py
```bash
from sqlalchemy import Column, Integer, String
from database import Base

class Atleta(Base):
    __tablename__ = 'atletas'
    id = Column(Integer, primary_key=True, index=True)
    nome = Column(String, index=True)
    cpf = Column(String, unique=True, index=True)
    centro_treinamento = Column(String)
    categoria = Column(String)
```
### schemas.py
```bash
from pydantic import BaseModel

class AtletaBase(BaseModel):
    nome: str
    cpf: str
    centro_treinamento: str
    categoria: str

class AtletaCreate(AtletaBase):
    pass

class AtletaResponse(AtletaBase):
    id: int

    class Config:
        orm_mode = True

class AtletaCustomResponse(BaseModel):
    nome: str
    centro_treinamento: str
    categoria: str
```
### database.py
```bash
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "postgresql://user:password@postgresserver/db"

engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```
### Execução
Instalar Dependências:
```bash
pip install fastapi sqlalchemy pydantic alembic fastapi-pagination uvicorn
```
### Rodar a API:
```bash
uvicorn main:app --reload
```
