```yml
#docker-compose.yml
version: "3"

services:
  db:
    environment:
    - POSTGRES_HOST_AUTH_METHOD=trust
    image: "postgres:12"
    container_name: "pg"
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data
      - ./word_data:/usr/share/dict
```

```console
mkdir word_data
cp /usr/share/dict/words word_data/words
docker-compose exec db bash
psql -U postgres -d query_plans -h 127.0.0.1
```
