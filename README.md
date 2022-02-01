#Monitorar servidor Python Flask com Prometheus e Grafana

#### post: https://dev.to/lucasgabz/monitorar-servidor-python-flask-com-prometheus-e-grafana-3ng7

Olá Dev, como vocês estão?

Bom, neste post irei mostrar a vocês como monitorar um app Python Flask. 
Afinal saber exatamente como nossa aplicação está funcionando é essencial!

Olha que coisa linda:
![Dashboard Exemplo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qktqbyb82dgnr0vk4dlt.png)

Vamos lá. Iremos utilizar o [Prometheus Flask Exporter](https://github.com/rycus86/prometheus_flask_exporter) e pra isso, dentro da nossa virtualenv iremos instalar ele usando:

```
pip install prometheus-flask-exporter
```


Em poucas linhas já podemos iniciar nosso monitoramento.

```
from flask import Flask
from prometheus_flask_exporter import PrometheusMetrics

app = Flask(__name__)
metrics = PrometheusMetrics(app)

@app.route('/')
def index():
    return 'Olá mundo!'

if __name__ == '__main__':
    app.run() 
```

O Prometheus utiliza do endereço '/metrics' para expor as métricas para o uso. 
Pode da uma olhada na sua aplicação 'localhost:5000/metrics'


Ta, tudo bem, chegamos até aqui, mas e agora? como vou fazer esse dashboard?

Vamos precisar do Prometheus e do Grafana, para isso já deixei separado um Docker Compose com tudo que precisamos

Dockerfile da nossa aplicação:

```
FROM python:3-alpine

ADD requirements.txt /requirements.txt

RUN pip install -r /requirements.txt

ADD app.py /app.py

ENV FLASK_APP /app.py

CMD flask run --host=0.0.0.0
```

Docker Compose:
```
version: '2'
services:

  # a sample app with metrics enabled
  app:
    image: python3-alpine
    build:
      context: app
    ports:
      - 5000:5000

  prometheus:
    image: prom/prometheus:v2.33.0
    volumes:
      - ./prometheus/config.yml:/etc/prometheus/prometheus.yml
    ports:
      - 9090:9090

  grafana:
    image: grafana/grafana:8.3.4
    ports:
      - 3000:3000
```

Tudo rodando, coisa linda. 
Vamos verificar!

Acessando a seguinte rota: `http://localhost:9090/targets#pool-app` veremos se o Prometheus já está buscando nossas métricas.



![prometheus app metrics](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x99nrzlfe176ansys14t.png)


Up!

Vamos olhar agora nosso Grafana e configurar algumas coisas.
Ao abrir a primeira vez `localhost:3000` vamos nos deparar com uma tela de login, o usuário e senha padrão do Grafana é `admin` e `admin`, após logar já vai pedir para que você mude a senha.


Certo, agora vamos adicionar o Data Source do Prometheus no Grafana
Para isso acesse o seguinte caminho `http://localhost:3000/datasources` e clique em 'Add data source' em seguida selecione o Prometheus(Provavelmente irá ser o primeiro da lista)


![prometheus config](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/homibr1jl2pjn3x4a9qu.png)

Vamos precisar somente alterar o caminho da url para http://prometheus:9090

Após isso é só ir até o final da pagina e clicar em Save & Test

Agora finalmente iremos criar nosso Dashboard, mas relaxa dev, é bem simples, já temos um template prontinho para isso chamado [Flask transaction](https://grafana.com/grafana/dashboards/9688)

Para importar esse template no Grafana basta seguir esta url `http://localhost:3000/dashboard/import` e no campo onde pede URL or ID colocamos o ID `9688`, clicamos em Load e depois em Import e Voilà!

Você já pode ver seu dashboard lindo, monitorando tudo da sua aplicação. =D

[Aqui está o repositório com todos os arquivos pronto deste post](https://github.com/biellsilva5/flask-com-grafana)

Não deixe de da uma olhadinha na documentação do [Prometheus Flask Exporter](https://github.com/rycus86/prometheus_flask_exporter) para se aprofundar mais. 