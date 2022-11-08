# Swarm-Vagrant

- Para executar é necessário que esteja instalado o libvirt
- Ambiente executado no Fedora 36

- Instalar plugin do vagrant para o libvirt
```vagrant plugin install vagrant-libvirt```

- Diferente do VirtualBox o libvirt para compartilhar o diretório /vagrant precisa que seja configurado o nfs.
- Execução em paralelo foi desativada, para que o manager seja criado antes dos demais managers e nodes do docker swarm.

