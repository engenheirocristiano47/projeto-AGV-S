# projeto-AGV-S

[ AGVs ]
    ↓ MQTT
[ Broker ]
    ↓
[ Backend - FastAPI ]
    ↓
[ PostgreSQL ]
    ↓
[ Dashboard Web + App Mobile ]



id (PK)
codigo
modelo
status
bateria
localizacao
ultima_comunicacao

id (PK)
origem
destino
prioridade
status
agv_id (FK)
inicio
fim


id (PK)
nome
descricao
trajeto_json
tempo_estimado


id (PK)
agv_id (FK)
descricao
nivel
data_hora
resolvido


id (PK)
agv_id (FK)
tipo
descricao
data_programada
status


id (PK)
tipo
descricao
data_hora


/agvs
/agvs/{id}
/ordens
/ordens/{id}
/rotas
/falhas
/manutencoes
/dashboard


agv_core/
 ├── app/
 │   ├── main.py
 │   ├── database.py
 │   ├── models.py
 │   ├── schemas.py
 │   ├── routes/
 │   │     ├── agvs.py
 │   │     ├── ordens.py
 │   │     ├── falhas.py
 │   │     ├── manutencao.py
 ├── mqtt/
 │     └── client.py
 └── requirements.txt


from fastapi import FastAPI
from routes import agvs, ordens, falhas, manutencao

app = FastAPI(title="AGV CORE - Stellantis")

app.include_router(agvs.router)
app.include_router(ordens.router)
app.include_router(falhas.router)
app.include_router(manutencao.router)

@app.get("/")
def root():
    return {"status": "AGV CORE rodando com sucesso 🚀"}


from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, declarative_base

DATABASE_URL = "postgresql://user:senha@localhost/agv_core"

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(bind=engine)

Base = declarative_base()


from sqlalchemy import Column, Integer, String, DateTime, Boolean, ForeignKey
from database import Base

class AGV(Base):
    __tablename__ = "agvs"

    id = Column(Integer, primary_key=True, index=True)
    codigo = Column(String)
    modelo = Column(String)
    status = Column(String)
    bateria = Column(Integer)
    localizacao = Column(String)
    ultima_comunicacao = Column(DateTime)


import paho.mqtt.client as mqtt

def on_message(client, userdata, msg):
    print(f"AGV -> {msg.topic}: {msg.payload.decode()}")

client = mqtt.Client()
client.connect("localhost", 1883)
client.subscribe("agv/#")

client.on_message = on_message
client.loop_forever()


pip install fastapi uvicorn sqlalchemy psycopg2-binary paho-mqtt
uvicorn app.main:app --reload





