VTI を使った routing-based IPsec VPN のために拡張<br>
(IKEv1, Aggressive Mode, NAT-T, PSK認証 のみでテスト)

NAT-T を使って同じグローバルIPアドレスから複数の VPN 終端装置がアクセスしてきた場合でも接続可能にする。

VPN 終端装置の識別にグローバルIPアドレスをユニークキーと見なせる環境では従来通り
 iptables を使用しない設定でも動作する。<br>
VPN 終端装置のグローバルIPアドレスが重複する場合は鍵交換の際に strongSwan が leftupdown スクリプトからソースアドレス・ポート番号を使ってその conn で使用する mark と同じ値を iptables で設定させる。

スクリプトで使いそうな変数はそれぞれ以下の通り。 (strongSwan の `src/_updown/_updown.in` より抜粋)

| 変数名 | 内容 |
| ------ | ----- |
| PLUTO_ME | is the IP address of our host. |
| PLUTO_PEER | is the IP address of our peer. |
| PLUTO_UDP_ENC | contains the remote UDP port in the case of ESP_IN_UDP encapsulation|
| PLUTO_MARK_IN | is an optional XFRM mark set on the inbound IPsec SA |

パッチは CentOS 7.2.1511 (kernel-3.10.0-327.28.2.el7.x86_64) との差分

