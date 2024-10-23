# LINUX
# proxy for curl wget git
proxy () {
  export ALL_PROXY="socks5://127.0.0.1:1089"
  export all_proxy="socks5://127.0.0.1:1089"
}

# unproxy
unproxy () {
  unset ALL_PROXY
  unset all_proxy
}


# WSL
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
# proxy for curl wget git npm apt
proxy () {
  export ALL_PROXY="http://$host_ip:7890"
  export all_proxy="http://$host_ip:7890"
 # echo -e "Acquire::http::Proxy \"http://$host_ip:10811\";" | sudo tee -a /etc/apt/apt.conf > /dev/null
 # echo -e "Acquire::https::Proxy \"http://$host_ip:10811\";" | sudo tee -a /etc/apt/apt.conf > /dev/null
}

# unproxy 
unproxy () {
  unset ALL_PROXY
  unset all_proxy
 # sudo sed -i -e '/Acquire::http::Proxy/d' /etc/apt/apt.conf
 # sudo sed -i -e '/Acquire::https::Proxy/d' /etc/apt/apt.conf
}

## reference
https://www.haoyep.com/posts/zsh-config-oh-my-zsh/
