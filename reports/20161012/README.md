###第1問  
##解答  
#関数解説
start()時にFDBを配列として複数用意する．スイッチとコントローラを接続する際，switch_ready()でdatapathごとのFDBを作る．それぞれのdatapathのFDBに対してpacket_in(), packet_out(), flow_mod()が行われる．これらの関数の動作は単一スイッチのみ対応のものと変わらないが，呼び出される際はdatapathを指定して行われる．  

#動作確認
用意されていたtrema.multi.confを用いて動作確認を行った．コントローラとスイッチの接続は