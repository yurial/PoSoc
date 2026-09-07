### 2.7 Канонизация

<a id="fmt-2.7"></a>

Канонический CBOR (ключи отсортированы); пары ключей лексикографически; самосвязи запрещены; $H$ — BLAKE3; округления порогов — $\lfloor x + 10^{-9} \rfloor$.

### 2.8 Record formats (норматив)

<a id="fmt-2.8"></a>

Record = `{type, t_sign, payload, sigs}`; sigs — массив `{pk, sig}`, **отсортированный по pk**. Signing payload = канонический CBOR `{type, t_sign, payload}` — каждый подписант подписывает одни и те же байты. Идентичность записи = $H(\mathrm{record})$ (полные канонические байты); дедупликация по содержимому.

| type | payload | sigs |
|---|---|---|
| `FRIEND` | `{x: pk, p: pk}` ($x < p$) | `[x, p]` |
| `FRIEND_REVOKE` | `{x: pk, p: pk, memo?: bstr ≤ 256}` (X→P) | `[x]` |
| `MEMBER_OF` / `MEMBER_OF_REVOKE` | `{group: ID_G, member: pk}` | `[signer]` |
| `MEMBER_OF_BATCH` | `{group: ID_G, members: [pk; ≤30]}` | `[signer]` |
| `DECLARE` / `DECLARE_REVOKE` | `{group: ID_H, declarant: pk}` | `[declarant]` |
| `L0_GENESIS` | `{descriptor: tstr ≤ 128, initiator: pk}` | `[initiator]` |
| `KEY_REVOKE` | `{memo?: bstr ≤ 256}` | `[self]` |

`MEMBER_OF` — «X member of G»: `member` — член $X$; подписант = $X$ — само-членство (его revoke — выход), подписант ≠ $X$ — подтверждение другого (его revoke — отзыв подтверждения). Подписант в payload не дублируется — он определяется подписью в sigs (по образцу `FRIEND {x, p}`: стороны — в payload, подписанты — в sigs).

Идентификаторы: $ID_G = H(\texttt{"L0"} \Vert \text{descriptor} \Vert pk_{initiator} \Vert t_{sign})$; $ID_H = H(\texttt{"GROUP"} \Vert \text{descriptor} \Vert pk_{founder})$, founder — pk первой (по $t_{sign}$) `DECLARE` (техническая роль без привилегий). Доменное разделение BLAKE3 — по тегам типов; tie-break $H(\mathrm{record})$ — от полных канонических байтов. Для `KEY_REVOKE` отзываемый ключ — единственный подписант (`self`); payload идентифицирующих полей не содержит ([ARCH-2.3.2](../concept/02_architecture.md#arch-2.3.2)).
