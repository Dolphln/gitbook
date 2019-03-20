在/etc/bashrc文件添加以下内容，将history记录到/var/log/command.log文件中

```
HISTDIR='/var/log/command.log'
if [ ! -f $HISTDIR ];then
touch $HISTDIR
chmod 720 $HISTDIR
fi
export HISTTIMEFORMAT="shell_history %F %T $HOSTNAME $(who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g') $(who am i|awk '{print $1}') ${USER} $(pwd|awk '{print $1}') \""
export PROMPT_COMMAND='history 1|tail -1|sed "s/^[ ]\+[0-9]\+  //"|sed "s/$/\"/">> /var/log/command.log'
```

source /etc/bashrc  使生效，或者重新登录都能让其生效

agent的配置文件/var/ossec/etc/ossec.conf，添加配置项

```
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/command.log</location>
</localfile>
```

/var/ossec/bin/ossec-control restart    重启ossec agent

日志格式如下：

```
shell_history 2019-03-20 15:58:33 centos6.5 10.100.1.10 root root /root "cat /var/log/command.log "
```

需要在ossec服务端添加decoder

/var/ossec/etc/decoder.xml

```
<decoder name="command-history">
  <prematch>^shell_history</prematch>
</decoder>
```

/var/ossec/rules/local\_rules.xml

```
<group name="local,history,">
<rule id="110003" level="5">
    <decoded_as>command-history</decoded_as>
    <description>test-command-history</description>
</rule>
</group>
```

最后效果   

![](/assets/shell-command-history.png)

