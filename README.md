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



## Reference

https://techlabs.net.br/integrando-mikrotik-com-syslog/
