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



## Reference

https://techlabs.net.br/integrando-mikrotik-com-syslog/
