### 2.7 Канонизация

Канонический CBOR (ключи отсортированы); пары ключей лексикографически; самосвязи запрещены; $H$ — BLAKE3; округления порогов — $\lfloor x + 10^{-9} \rfloor$.

### 2.8 Record formats (норматив)

Record = `{type, t_sign, payload, sigs}`; sigs — массив `{pk, sig}`, **отсортированный по pk**. Signing payload = канонический CBOR `{type, t_sign, payload}` — каждый подписант подписывает одни и те же байты. Идентичность записи = $H(\mathrm{record})$ (полные канонические байты); дедупликация по содержимому.

| type | payload | sigs |
|---|---|---|
| `LINK` | `{u: pk, v: pk}` ($u < v$) | `[u, v]` |
| `LINK_REVOKE` | `{from: pk, to: pk, memo?: bstr ≤ 256}` | `[from]` |
| `SELF_CONF` / `SELF_CONF_REVOKE` | `{group: ID_G, member: pk}` | `[member]` |
| `VOUCH` / `VOUCH_REVOKE` | `{group: ID_G, voucher: pk, member: pk}` | `[voucher]` |
| `VOUCH_BATCH` | `{group: ID_G, voucher: pk, members: [pk; ≤30]}` | `[voucher]` |
| `DECLARE` / `DECLARE_REVOKE` | `{group: ID_H, declarant: pk}` | `[declarant]` |
| `L0_GENESIS` | `{descriptor: tstr ≤ 128, initiator: pk}` | `[initiator]` |

Идентификаторы: $ID_G = H(\texttt{"L0"} \Vert \text{descriptor} \Vert pk_{initiator} \Vert t_{sign})$; $ID_H = H(\texttt{"GROUP"} \Vert \text{descriptor} \Vert pk_{founder})$, founder — pk первой (по $t_{sign}$) `DECLARE` (техническая роль без привилегий). Доменное разделение BLAKE3 — по тегам типов; tie-break $H(\mathrm{record})$ — от полных канонических байтов.