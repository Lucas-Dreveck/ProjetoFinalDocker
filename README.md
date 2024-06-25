<body style="text-align: justify">
    <h1>Como rodar o projeto:</h1>
    <div>
        <h2>Passo 1: Configurando o ambiente</h2>
        <ol>
            <li><p>Tenha (ou instale) o docker em sua máquina.</p></li>
            <li>
                <p>
                    Certifique-se que as portas que vão ser utilizadas no projeto estejam disponiveis para uso, caso não modifique-as.
                </p>
                <p style="margin-left: 20px">São elas:</p>
                <ul>
                    <li>9104 : mysql-exporter</li>
                    <li>6379 : redis</li>
                    <li>8080 : wordpress</li>
                    <li>9090 : prometheus</li>
                    <li>3000 : grafana</li>
                    <li>8081 : cadivisor</li>
                </ul>
            </li>
        </ol>
    </div>
    <hr/>
    <div>
        <h2>
            Passo 2: Clonar e acessar repositório do projeto
        </h2>
        <ol>
            <li>
                <p>
                    Em qualquer pasta do seu computador, de preferencia uma pasta que não possua caracteres especiais.
                </p>
                <p>Em seguida abra o git bash e digite o código a seguir:</p>
                <pre><code>git clone https://github.com/Lucas-Dreveck/ProjetoFinalDocker</code></pre>
            </li>
            <li>
                Com o projeto clonado em sua máquina, acesse a pasta do projeto:
                <pre><code>cd ProjetoFinalDocker</code></pre>
            </li>
        </ol>
    </div>
    <hr/>
    <div>
        <h2>Passo 3: Executar o projeto</h2>
        <ol>
            <li>
                Agora vamos iniciar nosso Docker Swarm para podermos iniciar o projeto projeto:
                <pre><code>docker swarm init</code></pre>
            </li>
            <li>
                Em seguida executaremos o comando:
                <pre><code>docker stack deploy -c docker-compose.yml "nome_stack"</code></pre>
                com este comando vamos iniciar nossos services que estão no arquivo .yml sendo eles:
                <ul>
                    <li>mysql - database</li>
                    <li>mysql-exporter - exporta os dados do mysql para serem acessados pelo prometheus</li>
                    <li>redis - usado para armazenar em cache as querys feitas pelo wordpress</li>
                    <li>wordpress - nosso site</li>
                    <li>prometheus - ferramenta de monitoramento</li>
                    <li>grafana - transforma os dados do prometheus em dashboards para melhor visualização</li>
                    <li>cadvisor - ferramenta para monitorar o docker e seus containers</li>
                </ul>
            </li>
        </ol>
    </div>
    <div>
        <h2>Passo 4: Configurar o wordpress</h2>
        <ol>
            <li>Abra seu navegador e acesse o link: <code>http://localhost:8080</code></li>
            <li>
                Ao acessar o wordpress, configure-o seguindo suas instruções, Obs: não se preocupe com os dados infomardos, todos são apenas para test.
            </li>
            <li>
                Finalizando o processo de configuração do worpress você será redirecionado a pagina de administrador:
                <img src="./md/image_wp_admin.png"/>                  
            </li>
            <li>
                Nesta pagina acesse a opção de plugins, você será redirecionado para está pagina onde estão os plugins instalados:
                <img src="./md/image_wp_plugins.png" />
            </li>
            <li>
                Ao lado da palavra Plugins no topo da pagina você terá a opção de Adicionar plugin, apertando nesse botão você será redirecionado para está pagina:
                <img src="./md/image_wp_adicionar_plugin.png"/>
            </li>
            <li>
                Pesquise na opção de busca o plugin: <code>Redis Object Cache</code>
                <img src="./md/image_wp_redis.png"/>
                Aperte na opção instalar, aguarda instalar e, após a instalação volte para a pagina de <code>Plugins Instalados</code>
            </li>
            <li>
                Abaixo dos plugins já instalados irá aparecer o Redis Object Cache, selecione a opção de Ativar(também pode aparecer como Configurações), ao acessar essa pagina podemos ver que não podemos nos conectar com o redis:
                <img src="./md/image_wp_erro_redis.png"/>
            </li>
        </ol>
    </div>
    <div>
        <h2>Corrigir erro de conexão entre o Redis e o Docker</h2>
        <ol>
            <li>
                Para isso vamos entrar no nosso terminal e digitar o comando <code>sudo su</code> para trocarmos para o usuário root do sistema, faça login com sua senha de root, em seguida digite o comando <code>docker ps</code>:
                Listamos todos os containers em execução, nessa lista vamos procurar o container que está instanciando o wordpress e copiarmos o CONTAINER ID dele.
            </li>
            <li>
                Vamos executar o comando <code>docker exec -it "ID DO SEU CONTAINER" bash</code>, assim vamos acessar a maquiná que está executando o wordpress.
            </li>
            <li>
                Em seguida vamos executar os comandos:
                <pre><code>apt update</code></pre>
                <pre><code>apt install nano</code></pre>
                Vamos atualizar o apt e em seguida baixar o aplicativo nano para que possamos editar o arquivo wp-config.php
            </li>
            <li>
                Após a conclusão do passo anterior digite o comando:
                <pre><code>nano wp-config.php</code></pre>
            </li>
            <li>
                Ao acessar a interface do nano procure a linha que diz <code>/* That's all, stop editing! Happy publishing. */</code>
                E adicione as linhas:
                <pre><code>define('WP_CACHE', true);<br>define('WP_REDIS_HOST', 'redis');<br>define('WP_REDIS_PORT', 6379);</code></pre>
            </li>
            <li>
                Salve o arquivo com o comando: "CTRL+O";<br>
                Feche o arquivo com o comando: "CTRL+X";
            </li>
            <li>
                Volte para o painel de plugins do wordpress, atualize a pagina apertando "F5" e veja se o Redis apareça como Acessível:
                <img src="./md/image_wp_redis_acessivel.png"/>
                Estando assim é só apertar no botão "Ativar o cache de objeto" e o Redis está configurado.
            </li>
        </ol>
    </div>
    <div>
        <h2>Passo 5: Acessar o Prometheus</h2>
        <ol>
            <li>
                Agora, acessaremos o link: <code>http://localhost:9090</code>, seremos redirecionados a tela inicial do Prometheus também sendo chamada de Graph, nesta tela podemos executar consultas de métricas.
            </li>
            <li>
                Podemos nele verificar metricas dos containers:
                <pre>container_cpu_system_seconds_total<br>container_fs_reads_total<br>container_fs_limit_bytes</pre>
                Também podemos visualizar as metricas do mysql:
                <pre>mysql_exporter_collector_success<br>mysql_global_status_connections<br>mysql_global_status_max_used_connections</pre>
                Entre várias outras métricas.
            </li>
        </ol>
    </div>
    <div>
        <h2>Passo 6: Dashboard do Grafana</h2>
        <ol>
            <li>
                Acessando a pagina: <code>http://localhost:3000</code> você será redirecionado para a pagina do grafana.
            </li>
            <li>
                Utilize o usuario admin e senha admin para entrar.
                <img src="./md/image_grafana_login.png"/>
            </li>
            <li>
                Pela dashboard vamos em Connections e iremos acessar a opção "Data sources".
                <img src="./md/image_grafana_datasource.png"/>
            </li>
            <li>
                Nessa aba, vamos apertar o botão "Add data source" e vamos buscar pelo Prometheus.
            </li>
            <li>
                Para configurar o data source do prometheus vamos seleciona-lo na lista abrindo assim suas configurações.
                Em Connection vamos adicionar a url do servidor do prometheus sendo ela <code>http://prometheus:9090</code>.
                <img src="./md/image_grafana_prometheus.png"/>
            </li>
            <li>Após adicionar sua url vamos no final da pagina e você ira apertar no botão "Save & test" assim podemos ir para a dashboards.</li>
            <li>Na dashboards do grafana vamos apertar em "Create Dashboard" e em seguida "Add visualization".</li>
            <li>No modal que apareceu vamos selecionar o Prometheus.</li>
            <li>Na opção Query, vamos seleiconar a métrica do Prometheus e apertar em run query.</li>
            <li>
                Assim será gerado um grafico e podemos apertar na opção "Apply" no canto superior direito
                <img src="./md/image_grafana_setqueries.png"/>
            </li>
        </ol>
    </div>
    <div>
        <h2>Passo Bônus: Cadvisor</h2>
        <ol>
            <li> Por curiosidade, caso queira olhar os graficos dos containers e do docker podemos acessar o cadvisor pela url <code>http://localhost:8081</code>.
        </ol>
    </div>
</body>