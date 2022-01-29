# rsyslog_and_mikrotik

# rsyslog

1 - Update e upgrade sua distro, no caso, utilizei Ubuntu 20.10

`apt -y update && apt -y upgrade`

2 - Habilite as portas no arquivo `/etc/rsyslog.conf` o protocolo usado sera UDP, entao, descomente as seguintes linhas: 

```
module(load="imudp")
input(type="imudp" port="514")
```

3 - Vamos criar um template que irá receber os logs remotos e ignorar os logs de localhost. Para uma melhor organização, vamos gerar o arquivo de log para cada Host/IP dentro do diretório `/var/log/routerOS/`

`nano /etc/rsyslog.d/routerOS.conf`

Adicione as seguintes linhas no arquivo criado:

```
$template RemoteHost,"/var/log/routerOS/%HOSTNAME%.log
:fromhost-ip, !isequal, "127.0.0.1" ?RemoteHost
```

Reinicie o serviço: 

`service rsyslog restart`

Verifique se o serviço está em execução:

`netstat -putan | grep 514`

Vamos configurar o logrotate para fazer a compressão dos logs: `/etc/logrotate.d/routerOS`

```
/var/log/routerOS/*.log {
        monthly
        rotate 12
        missingok
        notifempty
        sharedscripts
        postrotate
        /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
        endscript
}
```

Para testarmos vamos executar: 

`logrotate /etc/logrotate.d/routerOS --debug`

Caso apresente erro no arquivo, crie-o novamente utilizando outro console ssh como o Putty, pois pode haver problema de sintaxe com o Unix

Ok, tudo configurado do lado servidor rsyslog vamos configurar nosso mikrotik

# Mikrotik

1 - Agora, em nosso mikrotik vamos configurar o `logging`

![image](https://user-images.githubusercontent.com/37910997/151671058-de304ef1-44e9-43bd-ab0b-ca890fbbe3b2.png)

Lembre-se de trocar `IP_DO_SERVIDOR_RSYSLOG` pelo o IP do servidor que ira receber os logs vindo do mikrotik

2 - Depois de configurado o `Action` em `Logging` vamos configurar uma nova regra em `Rules`

![image](https://user-images.githubusercontent.com/37910997/151671168-f5dfc0bf-2f09-4ec5-8ba3-8056eba46c48.png)

3 - Feito isso, vamos para aba de `Firewall > NAT` e criar duas regras para `443,80`, voce pode adicionar outras regras para filtrar outras portas ou protocolos.

Em NAT vamos configurar as seguintes regras

![image](https://user-images.githubusercontent.com/37910997/151671281-a4ebbab8-faec-40ca-ba99-6c9cea77e3b6.png)

Adicione o seguinte: OBS: no meu caso, essas regras ficaram abaixo as demais ja configuradas :)

![image](https://user-images.githubusercontent.com/37910997/151671328-9c1b441b-81d9-475d-9650-f537cdf27b57.png)

![image](https://user-images.githubusercontent.com/37910997/151671337-86eb8017-0bf5-43c3-b438-44d92dedbd2b.png)

O mesmo sera feito para porta 443 

![image](https://user-images.githubusercontent.com/37910997/151671411-e449106b-1306-4632-8b2b-9e787e915441.png)

![image](https://user-images.githubusercontent.com/37910997/151671425-080fa12d-bda2-4bcd-bd72-74b80adec94c.png)

conforme falei anteriormente, outras regras podem ser adicionadas para filtrar ainda mais determinados acessos.

# Analisando o ambiente

Vamos imaginar o seguinte ambiente, voce precisa identificar quem acessou determinado site de porno em determinada hora por exemplo, poderia ser qualquer outra endereco de IP.

primeiro vamos exemplificar um pouco mais..

para identificar o endereco de IP de determinado site, eu posso dar um ping, como por exemplo:

![image](https://user-images.githubusercontent.com/37910997/151671946-142f0f88-e053-4e1a-9470-d27c53e81965.png)

Como voce pode observar, temos o range... se voce pegar o ip mostrado e filtrar no log, pode acontecer de voce nao encontrar :/

Entao, como resolver isso?

Bom,,,, voce pode pesquisar pelo range ou para facilitar ainda mais, basta acessar o seguinte site: https://www.nslookup.io/

![image](https://user-images.githubusercontent.com/37910997/151672007-8d92fd5a-26b7-4413-806a-777466881e6e.png)

observe que tem varios IPs, mas o range eh o mesmo, lembre-se que as vezes havera ranges diferentes :)

ok.... Vamos ver isso nos logs...

![image](https://user-images.githubusercontent.com/37910997/151672045-132b119c-ce2b-4dd8-8036-e7e1051b38b8.png)

otimo, vamos melhorar a visualizacao 

![image](https://user-images.githubusercontent.com/37910997/151672087-ecfebd9a-3763-4702-90d2-0b969eb9d941.png)
 
Entao, para filtrar de forma bem """"basicaa""" apenas utilizando o comando `cat` e `grep` podemos identificar o acesso realizado atraves de determinado usuario

![image](https://user-images.githubusercontent.com/37910997/151672142-fd457817-2203-4605-9396-e11351c23205.png)

logico que da para automatizar muito mais essas filtragens, isso eh apenas uma demonstracao e um passo a passo de como configurar um servidor de log com o mikrotik de forma funcional.


Ate a proxima :)


## Reference

https://techlabs.net.br/integrando-mikrotik-com-syslog/
