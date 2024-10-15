---
title: Instalación Material for MkDocs
---


Sitios web
----------
- [MkDocs](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/https:/)

Fuente
------
- https://www.youtube.com/watch?v=Q-YA_dA8C20

### Pre-requisitos
- Python
- Git y Github
- Visual Studio Code
    - Plugin MkDocs Syntax Highlight

#### Pasos
1. Crear el proyecto en github
    - Repositorio publico
    - .gitignore 
        - Pyhton
    - Licencia
        - GNU v3

2. Clonar el repositorio en local
    ```bash
    git clone git@github.com:terracenter/blog.git
    ```
    ```bash
    cd blog
    ```

3. Entorno Virtual Python
    ```bash
    sudo apt install python3.10-venv python3-pip
    ```    
    - Creación
        ```bash
        python3 -m venv venv 
        ```
    - Activación
        ```bash
         source venv/bin/activate
        ```

4. Instalación de mkdocs-material
    ```bash
    pip install mkdocs-material 
    ```

5. Creación de un MK-Docs Webiste
    ```bash
    mkdocs new .
    ```

6. Ejecutar el servidor en local
    ```bash
    mkdocs serve
    ```

7. Ver en el navegador
    ```bash
    http://127.0.0.1:8000/
    ```

8. Añadir Material for MkDocs
    - Editar el archivo `mkdocs.yml` y añadir:
       ```
       To do
       ``` 

9. Gitgub Actions
    - Crear carpeta de la raiz
        ```bash
        mkdir -p .github/workflows
        ``` 
    - Dentro de la carpeta `workflows`, crear el archivo `ci.yml`

        ```
        name: ci
        on:
        push:
            - master
            - main
        permissions:
        contents: write
        jobs:
        deploy:
            runs-on: ubuntu-latest
            steps:
            - uses: actions/checkout@v3
            - uses: actions/setup-python@v4
                with:
                python-version: 3.x
            - uses: actions/cache@v2
                with:
                key: ${{ github.ref }}
                path: .cache
            - run: pip install mkdocs-material
            - run: mkdocs gh-deploy --force
        ```
    - Añadir al stage
        ```bash
         git add .
         ```
    
    - Hacer el commit
        ```bash
         git commit -m $"Add Iniciando el Blog de documentación"
         ```
    - Hacer el push
        ```bash
        git push origin main
        ```

10. Ir a Github
    - Setting
        - pages
        


