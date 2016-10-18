###第1問  
##解答  
#関数解説
単一スイッチの場合と異なっていることとして，start()時にFDBを配列として複数用意している．そのため，スイッチとコントローラを接続する際，switch_ready()でdatapathごとのFDBを作る．それぞれのdatapathのFDBに対してpacket_in(), packet_out(), flow_mod()が行われる．これらの関数の動作は単一スイッチのみ対応のものと変わらないが，呼び出される際はdatapathを指定して行われる．  

#動作確認
用意されていたtrema.multi.confを少し簡略化して動作確認を行った．
```
vswitch('lsw1') { datapath_id 0x1 }
vswitch('lsw2') { datapath_id 0x2 }

vhost('host1')
vhost('host2')
vhost('host3')
vhost('host4')

link 'lsw1', 'host1'
link 'lsw1', 'host2'
link 'lsw2', 'host3'
link 'lsw2', 'host4'
```
この場合のコントローラとスイッチの接続は画像のようになる．
![topology](https://github.com/handai-trema/learning-switch-Shu-NISHIKORI/tree/master/reports/20161012/topology.jpg "topology")

以下のように動作させた．  
1. host1からhost2へパケットを送信する．  
2. host2からhost1へパケットを送信する．  
3. host1からhost2へパケットを送信する．  
結果は以下のようになった．
```
$ ./bin/trema send_packets --source host1 --dest host2
$ ./bin/trema show_stats host1
Packets sent:
  192.168.0.1 -> 192.168.0.2 = 1 packet
$ ./bin/trema show_stats host2
Packets received:
  192.168.0.1 -> 192.168.0.2 = 1 packet
$ ./bin/trema send_packets --source host2 --dest host1
$ ./bin/trema show_stats host1
Packets sent:
  192.168.0.1 -> 192.168.0.2 = 1 packet
  192.168.0.2 -> 192.168.0.1 = 1 packet
$ ./bin/trema show_stats host2
Packets received:
  192.168.0.1 -> 192.168.0.2 = 1 packet
  192.168.0.2 -> 192.168.0.1 = 1 packet
$ ./bin/trema send_packets --source host1 --dest host2
$ ./bin/trema show_stats host1
Packets sent:
  192.168.0.1 -> 192.168.0.2 = 2 packet
  192.168.0.2 -> 192.168.0.1 = 1 packet
$ ./bin/trema show_stats host2
Packets received:
  192.168.0.1 -> 192.168.0.2 = 2 packet
  192.168.0.2 -> 192.168.0.1 = 1 packet
```
パケットの送受信関係から，きちんと転送されていることを確認した． 
コントローラの動作については，各手順で画像のように動作している．
![flow](https://github.com/handai-trema/learning-switch-Shu-NISHIKORI/tree/master/reports/20161012/flow.jpg "flow")

同様に，スイッチ2についても動作実験を行った．  
1. host3からhost4へパケットを送信する．  
2. host4からhost3へパケットを送信する．  
3. host3からhost4へパケットを送信する．  
結果は以下のようになり，きちんと動作していることを確認した．  
```
$ ./bin/trema send_packets --source host3 --dest host4
$ ./bin/trema show_stats host3
Packets sent:
  192.168.0.3 -> 192.168.0.4 = 1 packet
$ ./bin/trema show_stats host4
Packets received:
  192.168.0.3 -> 192.168.0.4 = 1 packet
$ ./bin/trema send_packets --source host4 --dest host3
$ ./bin/trema show_stats host3
Packets sent:
  192.168.0.3 -> 192.168.0.4 = 1 packet
  192.168.0.4 -> 192.168.0.3 = 1 packet
$ ./bin/trema show_stats host4
Packets received:
  192.168.0.3 -> 192.168.0.4 = 1 packet
  192.168.0.4 -> 192.168.0.3 = 1 packet
$ ./bin/trema send_packets --source host3 --dest host4
$ ./bin/trema show_stats host3
Packets sent:
  192.168.0.3 -> 192.168.0.4 = 2 packet
  192.168.0.4 -> 192.168.0.3 = 1 packet
$ ./bin/trema show_stats host4
Packets received:
  192.168.0.3 -> 192.168.0.4 = 2 packet
  192.168.0.4 -> 192.168.0.3 = 1 packet
```

最後に，host1からhost4に送信する場合を考える．スイッチ1からコントローラにpacket_in()を要請した後，コントローラはスイッチ1にflood命令を出すが，スイッチ1にhost4は接続されていないため，到達することは無い（画像参照）．よって，確かにスイッチごとに独立していることが分かる．
![error](https://github.com/handai-trema/learning-switch-Shu-NISHIKORI/tree/master/reports/20161012/error.jpg "error")