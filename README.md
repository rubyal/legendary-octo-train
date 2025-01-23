# Proyecto: Despliegue de PostgreSQL y Jupyter con Docker

Este proyecto detalla los pasos para desplegar PostgreSQL y Jupyter usando Docker, así como la creación de una base de datos, esquema, e inserción de datos desde Jupyter.

## Tabla de Contenidos
1. [Requisitos Previos](#requisitos-previos)
2. [Instrucciones de Despliegue](#instrucciones-de-despliegue)
     - [1. Configuración de PostgreSQL y Jupyter con Docker]
3. [Creación de la Base de Datos y Esquema](#creación-de-la-base-de-datos-y-esquema)
4. [Inserción de Datos desde Jupyter](#inserción-de-datos-desde-jupyter)
5. [Problemas Comunes y Soluciones](#problemas-comunes-y-soluciones)
6. [Referencias](#referencias)

---

## Requisitos Previos

Asegúrate de tener instalado:
- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- Conexión a internet para descargar imágenes de Docker
- Acceso a un editor de texto para crear y modificar archivos (`vim`, `nano`, `VSCode`, etc.)
---

## Instrucciones de Despliegue

### 1. Configuración de PostgreSQL y Jupyter con Docker

1. Crea un archivo `vi docker-compose.yaml` con el siguiente contenido:

```yaml
version: '3.7'
services:
  postgres:
    image: postgres:15
    container_name: postgres
    hostname: ****
    environment:
      POSTGRES_USER: ruby
      POSTGRES_PASSWORD: ****
      POSTGRES_DB: pokemondb
    ports:
      - "5432:5432"
    networks:
      - kafka-net
  jupyter:
    image: jupyter/base-notebook:latest
    container_name: jupyter_notebook
    ports:
      - "8888:8888"
    volumes:
      - ./notebooks:/home/jovyan/work
    environment:
      - JUPYTER_TOKEN=********
    networks:
      - kafka-net
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: ruby@example.com
      PGADMIN_DEFAULT_PASSWORD: ********
    ports:
      - "5050:80"
    depends_on:
      - postgres
    networks:
      - kafka-net
    volumes:
      - pgadmin-data:/var/lib/pgadmin
networks:
  kafka-net:
    driver: bridge
volumes:
  pgadmin-data:
```

2. Levanta el contenedor con el comando:
    ```bash
    docker-compose up -d
    ```

3. Verifica que el contenedor esté corriendo:
    ```bash
    docker ps
    ```
4. Accede al postgres desde tu navegador en [http://localhost:5050](http://localhost:5050). Usa el username y password que aparece en los logs del contenedor:
    ```bash
    docker logs jupyter-container
    ```
5. Accede al Jupyter Notebook desde tu navegador en [http://localhost:8888](http://localhost:8888). Usa el token que aparece en los logs del contenedor:
    ```bash
    docker logs jupyter-container
    ```

![image](https://github.com/user-attachments/assets/c7802f35-b203-49ff-bbd3-ef4a2d343a7b)
![image](https://github.com/user-attachments/assets/0d6c8f4b-9307-4b69-a56d-7aca9fb3016e)

![image](https://github.com/user-attachments/assets/9020bc58-7707-4ae9-81dd-f4dd532e8380)

## Creación de la Base de Datos y Esquema
```
#### Connection details
host = "*****"
port = "5432"
database = "pokemondb"
user = "ruby"
password = "****"  

#### Connect to PostgreSQL and create schema and table
try:
    conn = psycopg2.connect(
        host=host,
        port=port,
        database=database,
        user=user,
        password=password
    )
    conn.autocommit = True
    cursor = conn.cursor()
    
    # Create schema and table
    cursor.execute("""
    CREATE SCHEMA IF NOT EXISTS pokemon_data;
    
    CREATE TABLE IF NOT EXISTS pokemon_data.pokemon (
        number INT,
        name VARCHAR(100),
        type1 VARCHAR(50),
        type2 VARCHAR(50),
        total INT,
        hp INT,
        attack INT,
        defense INT,
        sp_attack INT,
        sp_defense INT,
        speed INT,
        generation INT,
        legendary BOOLEAN,
        PRIMARY KEY (number, name)
    );
    """)
    print("Schema and table created successfully!")
    
    cursor.close()
    conn.close()
    
except Exception as e:
    print(f"Error: {e}")
```


![image](https://github.com/user-attachments/assets/8b73a219-cab6-45ee-aba9-634ecc3b6dd7)
![image](https://github.com/user-attachments/assets/08563bc7-8a3d-4399-b7c1-fdf882cd1131)


## Inserción de Datos desde Jupyter
```
# Load the dataset
file_path = "Pokemon.csv"

df = pd.read_csv(file_path)

# Connection details
host = "****"
port = "5432"
database = "pokemondb"
user = "ruby"
password = "****" 

# Connect to the database and insert data
try:
    conn = psycopg2.connect(
        host=host,
        port=port,
        database=database,
        user=user,
        password=password
    )
    conn.autocommit = True
    cursor = conn.cursor()
    
    # Insert data row by row
    for index, row in df.iterrows():
        cursor.execute("""
        INSERT INTO pokemon_data.pokemon (
            number, name, type1, type2, total, hp, attack, defense, sp_attack, sp_defense, speed, generation, legendary
        ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        ON CONFLICT (number, name) DO NOTHING;
        """, (
            row['number'], row['name'], row['type1'], 
            row['type2'] if pd.notnull(row['type2']) else None,  # Handle NULL values
            row['total'], row['hp'], row['attack'], row['defense'], 
            row['sp_attack'], row['sp_defense'], row['speed'], 
            row['generation'], row['legendary']
        ))

    print("Data inserted successfully!")
    cursor.close()
    conn.close()
    
except Exception as e:
    print(f"Error: {e}")
```
![image](https://github.com/user-attachments/assets/90d45b23-eb14-4d2d-8244-69b89bdd4221)


![image](https://github.com/user-attachments/assets/c3b6c6d3-b69f-4514-bd1f-d91173a4016f)


![image](https://github.com/user-attachments/assets/0049b1c5-fe1e-43ae-8d99-fadd0c0c8da7)


![image](https://github.com/user-attachments/assets/7400f6c1-84bf-4171-a33c-db4512214335)



![image](https://github.com/user-attachments/assets/006a0105-a1c7-4c84-b4b1-cbd3f7c7026c)



![image](https://github.com/user-attachments/assets/afbacb95-750e-4abf-a42d-e4c8abd1ae0c)

## Problemas Comunes y Soluciones

### Error: `Connection refused` al conectar desde Jupyter a PostgreSQL
- Asegúrate de que los puertos están mapeados correctamente y el contenedor de PostgreSQL está en ejecución.

### El token de Jupyter no aparece
- Revisa los logs del contenedor de Jupyter usando `docker logs`.

---

## Referencias
- [Documentación de Docker Compose](https://docs.docker.com/compose/)
- [psycopg2](https://www.psycopg.org/docs/)
- [Jupyter Notebook Docker](https://hub.docker.com/r/jupyter/base-notebook)
