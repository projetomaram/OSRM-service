# OSRM-service

## Descrição
Este serviço é dividido em dois serviços menores, porém ambos são derivados da mesma aplicação, o [Open Source Routing Machine](https://project-osrm.org/) (OSRM), um é o frontend e outro é o backend, cada serviço possui uma imagem Docker. Vale ressaltar que o frontend não é semelhante ao da aplicação original, ele foi modificado para evitar consultas externas não utilizadas, seu layout também foi modificado para se encaixar ao projeto, como também foi adicionado uma funcionalidade de poder ver um carrinho percorrendo a rota selecionada, todas as modificações feitas são indexadas no arquivo **alteracoes-front.txt**.

## Funcionamento
Por senso, e por escolha de implementação, o frontend é quase que totalmente dependente do backend, sem ele não é possível realizar consulta de rotas no mapa. Idealmente o frontend é mapeado para funcionar na porta 9966 e o backend na porta 5000, quando são executados separados do resto dos serviços. Por padrão, na imagem é escolhido a região sul do Brasil como base para o backend, caso seja necessário alterar essa região, deve-se alterar em código no arquivo **Dockerfile.backend** o argumento *PBF_URL* e então realizar o **build** da imagem novamente. Por questões de implementação, a aplicação como um todo deve ser executada em um ambiente Kubernetes, pois a execução individual utilizando `docker run` faz com que o frontend não encontre corretamente o caminho do backend.

Para que seja possível a execução individual dos serviços do resto da aplicação existe duas formas, por meio do manifesto Kubernetes **osrm.yaml**, ou então alterando o arquivo `front-mod/src/leaflet-options.src`, mais especificamente a região:
```
  services: [{
    label: process.env.OSRM_LABEL || 'Car (fastest)',
    path: '/route/v1'
  }],
```
Alterando o path para o endereço do backend, por exemplo `http://localhost:5000/route/v1`, caso esteja sendo executado na mesma máquina do backend, e realizando o build novamente da imagem Docker.

## Exemplo de uso individual
Em caso de necessidade de executar os serviços separados utilizando `docker run` é  necessário primeiro executar o backend para isso basta executar o comando:
```bash
docker run -d -p 5000:5000 projetomaram/robot-project:osrm-backend
```
Com o backend em execução e o endereço do backend alterado para `localhost:5000`, é necessário que se realize o build da imagem:
```bash
docker build -t frontend-personalizado -f Dockerfile.frontend .
```
Para que então possa ser executado o frontend:
```bash
docker run -d -p 9966:9966 frontend-personalizado
```
Para este caso o frontend irá estar disponível no endereço `http://localhost:9966`.

Como dito anteriormente é possível executar tudo sem necessidade de alterar o endereço de backend, para isso utiliza-se o ambiente Kubernetes. Com o cluster já em execução (esse passo é mostrado no [Guia de execução](https://github.com/projetomaram/Microsservicos-Aplicado-em-Robotica-Autonoma-Movel?tab=readme-ov-file#guia-de-execução)), basta executar o comando:
```bash
kubectl create -f osrm.yaml
```
Quando todos os serviços estiverem execução, o frontend irá estar disponível no endereço `http://localhost`.
