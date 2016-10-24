VTI を使った routing-based IPsec VPN のために拡張<br>
(IKEv1, Aggressive Mode, NAT-T, PSK認証 のみでテスト)

NAT-T を使って同じグローバルIPアドレスから複数の VPN 終端装置がアクセスしてきた場合でも接続可能にする。

鍵交換の際に strongSwan が leftupdown スクリプトからソースアドレス・ポート番号を使ってその conn で使用する mark と同じ値を iptables で設定させる。

```
## ソースアドレス・ポートで mark を設定する
# iptables -t mangle -I PREROUTING -s ${PLUTO_PEER} -p udp --sport ${PLUTO_UDP_ENC} -j MARK --set-mark ${PLUTO_MARK_IN%%/*}

## strongSwan を動かす Linux 自身への着信がエラーにならないように 0 に戻す
## (conntrack には送信時に mark 0 で記録されているため icmp_rcv などから flow を探す時に見つけられず受信できないようだ)
# iptables -t mangle -I INPUT -i vti+ -j MARK --set-mark 0

## MSS 書き換え
# iptables -I FORWARD -i vti+ -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
# iptables -I FORWARD -o vti+ -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
```

スクリプトで使いそうな変数はそれぞれ以下の通り。 (strongSwan の `src/_updown/_updown.in` より抜粋)

| 変数名 | 内容 |
| ------ | ----- |
| PLUTO_ME | is the IP address of our host. |
| PLUTO_PEER | is the IP address of our peer. |
| PLUTO_UDP_ENC | contains the remote UDP port in the case of ESP_IN_UDP encapsulation|
| PLUTO_MARK_IN | is an optional XFRM mark set on the inbound IPsec SA |

パッチは CentOS 7.2.1511 (kernel-3.10.0-327.28.2.el7.x86_64) との差分

