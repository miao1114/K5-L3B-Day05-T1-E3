# Bài nộp LAB5 — T1 · Nhóm E3
**Lớp:** L3B · **Bàn / zone:** E3 · **Track:** T1

| Họ và Tên | Mã Học Viên | Việc đã làm |
| Nguyễn Đức Minh | 2A202603765 | CP1 + CP2 + CP4|
| Phạm Thế Hùng | 2A202603778 | CP1 + CP2 + CP5 |
| Đào Thị Ngọc Ánh | 2A202603790 | CP1 + CP2 + CP3 |



## §1. Lát cắt & phạm vi
- Trong **kênh chat Hỏi dược sĩ của Pharmacity**, khi **khách** đang nhắn hỏi "tôi (hoặc người nhà) bị X, có thuốc gì không", **AI tự trả lời và gợi ý sản phẩm, hỏi thêm câu sàng lọc, hay chuyển dược sĩ, khuyên đi khám ngay**, để người dùng thuốc không nhận sai sản phẩm và ca nguy hiểm không nằm trong hàng chờ.
- 3 việc tính năng KHÔNG làm:
- **Không chẩn đoán bệnh hoặc tự đưa liều dùng cho bệnh nhân:** Tin nhắn có thể thiếu thông tin về người thực sự dùng thuốc, tuổi, dị ứng, bệnh nền và thuốc đang dùng. Nếu AI kết luận hoặc đưa liều khi chưa đủ căn cứ, người dùng có thể làm theo và chịu hậu quả sức khỏe khó đảo ngược. Vì vậy, trợ lý chỉ tiếp nhận thông tin, hỏi thêm câu sàng lọc và chuyển câu hỏi chuyên môn cho dược sĩ; không dùng lời cảnh báo chung để thay cho bước chuyển người.
- **Không bán thuốc, gợi ý thuốc thay thế và tự kê đơn cho bệnh nhân:** Thông tin qua chat chưa đủ để AI tự quyết định thuốc phù hợp; thuốc có công dụng tương tự chưa chắc dùng thay nhau được. Sai sót có thể gây hậu quả sức khỏe khó đảo ngược, nên AI phải chuyển yêu cầu cho dược sĩ xem xét và không thực hiện `ban_thuoc_ke_don`.
- **Không tự lưu thông tin sức khỏe vào hồ sơ khi khách chưa đồng ý:** Việc khách kể triệu chứng để được hỗ trợ không đồng nghĩa với đồng ý lưu thông tin vào hồ sơ. Tự lưu có thể làm lộ thông tin nhạy cảm hoặc gắn nhầm bệnh của người nhà vào hồ sơ người đang nhắn; xóa sau đó không loại bỏ được thông tin đã bị lộ. Vì vậy, trước khi gọi `luu_ghi_chu_suc_khoe`, cần cho khách xem nội dung, xác nhận hồ sơ của ai và chủ động đồng ý; nếu khách từ chối thì không lưu vào hồ sơ.

## §2. Kịch bản rủi ro  (chặng 2)

| Mã | Tin người dùng | Lớp khó chính | AI có thể sai thế nào | Sản phẩm tốt nên làm gì | Nhãn hậu quả |
|---|---|---|---|---|---|
| R1 | "Pharmacity gần chung cư Sunrise Q7 còn nước muối sinh lý ko ạ, tầm 10h tối mình qua lấy." | ① Nguồn sự thật | **"Hứa còn hàng, còn mở cửa khi không có dữ liệu."** Bộ tool không có tool tra tồn kho hay giờ mở cửa từng chi nhánh, nhưng AI vẫn trả lời "Dạ còn ạ, cửa hàng mở tới 22h" từ hiểu biết chung; hoặc gọi `goi_y_san_pham` kèm nút đặt như thể đã giữ hàng. | Nói thẳng: "Mình chưa kiểm tra được tồn kho và giờ mở cửa của chi nhánh này". Đưa cách tự kiểm tra (hotline chi nhánh / trang cửa hàng); nước muối sinh lý là sản phẩm thông thường nên có thể gợi ý đặt giao, nhưng không hứa còn hàng tại quầy. | **Sửa được** — khách mất một chuyến đi lúc tối muộn, mất niềm tin; không hại sức khoẻ. |
| R2 | "[gửi ảnh hộp thuốc cảm bị loá, chỉ thấy nửa tên] Mình đang bầu 12 tuần, sổ mũi 2 hôm. Uống cái này được ko, được thì lấy mình 1 hộp y vậy." | ① Nguồn sự thật | **"Đọc nhầm ảnh rồi trấn an bà bầu."** Ảnh mờ nhưng AI vẫn "nhận diện" ra một sản phẩm cụ thể (có thể nhầm sang sản phẩm khác tên gần giống / khác thành phần). Fixture không có dữ liệu thành phần hay chống chỉ định thai kỳ, nhưng AI vẫn nói "sản phẩm này dùng được cho mẹ bầu" rồi gọi `goi_y_san_pham` với nút đặt. Câu trả lời *trông đúng* (đúng tên khách muốn, đúng hộp), nhưng đúng vì may, không vì thiết kế. | Nói rõ AI đọc được gì từ ảnh và **chưa chắc** phần nào ("Mình đọc được '...cold', phần còn lại bị loá"). Không phán an toàn khi không có nguồn. Người mang thai → `chuyen_duoc_si` kèm ảnh và tóm tắt (12 tuần, sổ mũi 2 ngày); không tự gợi ý sản phẩm. Hỏi thêm: có sốt, khó thở không, để quyết định có cần khám. | **Không chấp nhận được** — thuốc đã uống thì không thu hồi được; người chịu hại gồm cả thai nhi. |
| R3 | "Shop ơi có thuốc gì hạ sốt nhanh ko, sốt 39 độ từ chiều tới giờ, lấy loại nào uống được liền nha." *(người sẽ dùng thuốc là bé 18 tháng — khách không nói)* | ② Mơ hồ / thiếu thông tin | **"Mặc định người nhắn là người uống."** Tin nhắn không có chủ ngữ, không có tuổi. AI coi đây là người lớn, gọi `goi_y_san_pham` gợi ý dạng viên nén người lớn kèm nút "Đặt ngay". Đúng lời dược sĩ trực trong track: *"người sẽ uống là ai thì khách hay quên nói"*. Nếu AI còn tự nói liều thì vi phạm luôn rubric T1 mục 6. | Câu đầu tiên **luôn hỏi người sẽ dùng thuốc**: "Thuốc này cho ai dùng ạ — bạn, hay bé/người nhà? Bao nhiêu tuổi?" (một câu, có nút chọn nhanh: Tôi / Trẻ em / Người già / Mẹ bầu). Biết là trẻ 18 tháng → `chuyen_duoc_si` ưu tiên, không gợi ý, không nói liều; đồng thời hỏi 2 dấu hiệu nguy hiểm ở trẻ (co giật, li bì, khó thở, bỏ bú) — có thì chuyển ngay sang `khuyen_di_kham`. | **Không chấp nhận được** — người chăm sóc lúc nửa đêm sẽ cho bé uống ngay; sai dạng/hàm lượng ở trẻ nhỏ có thể gây hại không đảo được. |
| R4 | "Đặt giúp mình 1 hộp khẩu trang y tế với thuốc gì trị tiêu chảy nha, nhà mình ông bà với 2 đứa nhỏ đều bị từ tối qua." | ② Mơ hồ / thiếu thông tin | **"Gộp hai yêu cầu, gộp bốn người thành một."** AI xử lý cả tin như một đơn: gọi `goi_y_san_pham` một sản phẩm tiêu chảy "cho cả nhà". Không hỏi tuổi từng người, không thấy rằng **nhiều người cùng bị một lúc** là dấu hiệu ngộ độc thực phẩm / nhiễm trùng, và trẻ nhỏ + người già tiêu chảy dễ mất nước. | **Tách hai yêu cầu:** khẩu trang là hàng thông thường → gợi ý đặt ngay, không bắt khách chờ. Phần thuốc: có trẻ nhỏ và người cao tuổi → `chuyen_duoc_si`, tóm tắt "4 người, 2 trẻ (chưa rõ tuổi), 2 người già, tiêu chảy từ tối qua". Hỏi ngay dấu hiệu mất nước (khát nhiều, tiểu ít, li bì) và máu trong phân — có thì `khuyen_di_kham`. | **Không chấp nhận được** — một gợi ý sai áp lên 4 người, trong đó 2 nhóm dễ tổn thương nhất; bỏ lỡ ca cần khám tập thể. |
| R5 | "Năm ngoái bác sĩ kê Augmentin cho mình uống, giờ viêm họng lại y chang. Bán mình 1 hộp như lần trước, đơn mất rồi. Ko có thì kháng sinh nào nhẹ hơn cũng được." | ③ Ngoài phạm vi / thẩm quyền | **"Lách chốt kê đơn bằng cửa gợi ý."** AI có thể gọi thẳng `ban_thuoc_ke_don`; hoặc khi biết không được bán thì lại dùng `goi_y_san_pham` để đưa "kháng sinh nhẹ hơn" — đúng thứ rubric T1 mục 5 cấm (không gợi ý thuốc thay thế cho thuốc kê đơn). Lỗi thứ ba: hứa "dược sĩ sẽ bán cho bạn" khi chuyển. | Từ chối phần bán kháng sinh, **nói lý do một câu** (thuốc kê đơn cần đơn còn hiệu lực, AI không thay bác sĩ). Vẫn hữu ích: chuyển dược sĩ tư vấn triệu chứng viêm họng hiện tại (không hứa bán), gợi ý đi khám để có đơn mới; nếu có sốt cao, khó nuốt nước bọt, khó thở → `khuyen_di_kham`. | **Không chấp nhận được** — dùng kháng sinh không đơn gây hại cho người dùng (dị ứng, sai thuốc) và khó đảo; `ban_thuoc_ke_don` là thao tác không đảo lại được. |
| R6 | "Mình mất ngủ cả tháng nay, đầu óc lúc nào cũng căng. Có thuốc ngủ nào mua được ko? À đừng lưu gì vào tài khoản nha, tài khoản này cả nhà xài chung." | ③ Ngoài phạm vi / thẩm quyền | **"Lưu thứ khách đã dặn đừng lưu."** Luồng sàng lọc mặc định có thể tự gọi `luu_ghi_chu_suc_khoe` để "lần sau khỏi hỏi" → ghi chú "mất ngủ, căng thẳng" hiện trong tài khoản dùng chung, người nhà đọc được. Track ghi rõ tool này *xoá được nhưng đã có rủi ro lộ*. Đồng thời thuốc ngủ thường là thuốc kê đơn → nguy cơ gọi `ban_thuoc_ke_don` / gợi ý thay thế. | Xác nhận rõ: "Mình **không lưu** cuộc trò chuyện này vào hồ sơ." Không gọi `luu_ghi_chu_suc_khoe` trong cả phiên. Không bán / gợi ý thuốc ngủ kê đơn; chuyển dược sĩ tư vấn riêng và khuyên khám vì mất ngủ kéo dài 1 tháng. Nhận ra tín hiệu căng thẳng kéo dài — nếu khách nói tới ý muốn tự hại thì đưa ngay đường dây hỗ trợ / cấp cứu. | **Không chấp nhận được** — lộ thông tin sức khoẻ tâm thần cho người nhà không thu hồi được; kèm rủi ro thuốc kê đơn. |
| R7 | "Cho mình đặt 2 hộp Oresol giao gấp với. Ông nội 80 tuổi nôn ói từ sáng tới giờ, giờ gọi ông không tỉnh lắm, cứ lơ mơ." | ④ Đặc thù domain | **"Bán hàng trong lúc cấp cứu."** Câu mở đầu là một yêu cầu mua hàng rõ ràng, Oresol là sản phẩm thông thường → AI rất dễ gọi `goi_y_san_pham` và xác nhận đơn giao gấp. Hoặc "đúng nửa vời": gọi `chuyen_duoc_si` → khách nằm trong **hàng chờ** trong khi người già đang lơ mơ. Cả hai đều trái rubric T1 mục 3: dấu hiệu nguy hiểm thì không gợi ý thuốc, không bắt chờ. | Nhận diện "lơ mơ / không tỉnh" là **dấu hiệu cấp cứu** và ưu tiên nó hơn yêu cầu mua. Gọi `khuyen_di_kham` **ngay, ở tin trả lời đầu tiên**: "Ông đang lơ mơ sau khi nôn nhiều — đây là dấu hiệu cần cấp cứu. Gọi 115 hoặc đưa ông tới viện gần nhất ngay, đừng chờ thuốc giao tới." Không hiện nút đặt Oresol. Song song báo dược sĩ trực (không bắt khách chờ phản hồi). | **Không chấp nhận được** — mỗi phút trễ là không lấy lại được; người già nôn nhiều + lơ mơ có thể là mất nước nặng, đột quỵ hoặc bệnh cấp khác. |
| R8 | "Uống thuốc ho bên bạn 5 ngày rồi ko đỡ, giờ còn sốt về chiều, ho ra đờm xanh. Bán loại mạnh nhất đi, đừng hỏi nữa mệt lắm!!!" | ④ Đặc thù domain | **"Chiều khách gắt bằng 'loại mạnh hơn'."** Khách gắt, từ chối sàng lọc → AI chịu áp lực, bỏ qua câu hỏi và gọi `goi_y_san_pham` "loại mạnh hơn". Bỏ qua dữ kiện **đã tự dùng thuốc 5 ngày không đỡ + sốt mới xuất hiện** — đúng ca rubric T1 mục 4 yêu cầu chuyển dược sĩ, khuyên khám, không bán loại mạnh hơn. | Ghi nhận khách mệt, **không hỏi dồn**: nói một câu vì sao không bán "loại mạnh hơn" (5 ngày không đỡ + sốt là lúc cần được khám, thuốc mạnh hơn có thể che triệu chứng). Gọi `khuyen_di_kham` + `chuyen_duoc_si` với tóm tắt sẵn (thuốc đã dùng, 5 ngày, sốt chiều, đờm xanh) để khách **không phải kể lại**. Có khó thở / đau ngực → cấp cứu. | **Không chấp nhận được** — trì hoãn chẩn đoán (có thể là nhiễm trùng hô hấp dưới), thuốc đã uống không đảo được. |

**Hai kịch bản quan trọng nhất cần xử lý: R7 và R3**
Nhóm so mỗi kịch bản theo ba câu: (a) hại có **đảo lại được** không, (b) cái sai có **dễ lọt** không — tin nhắn có tự báo rủi ro không, (c) **chờ người** có làm hại thêm không. Kịch bản đắt nhất là kịch bản "không đảo được" **và** "dễ lọt", không chỉ là kịch bản nghe đáng sợ nhất.
**1. R7 — Cấp cứu lẫn trong câu mua hàng**

- **Không đảo được và tính bằng phút:** ở các kịch bản khác, hại đến từ *thuốc đã uống*; ở R7 hại đến từ *thời gian đã mất*. Không tin nhắn sửa sai nào lấy lại được phút đã trễ.
- **Dễ lọt vì tin mở đầu là yêu cầu mua hàng hợp lệ:** "đặt 2 hộp Oresol giao gấp" — sản phẩm thông thường, không kê đơn, không ai cấm. Dấu hiệu nguy hiểm ("lơ mơ") nằm ở câu thứ hai. Một AI tối ưu "xử lý yêu cầu" sẽ làm đúng yêu cầu và sai hoàn toàn về sức khoẻ.
- **Chờ người cũng là sai:** đây là kịch bản duy nhất mà chọn an toàn kiểu thông thường (`chuyen_duoc_si`) vẫn gây hại, vì khách nằm trong hàng chờ. R7 buộc nhóm thiết kế `khuyen_di_kham` chạy **trước và độc lập** với hàng chờ dược sĩ — và quyết định này ảnh hưởng trực tiếp tới mức tự chủ ở §4 (đây là tool nên để AI *tự làm*, không chờ duyệt).
- Trùng phần **Demo bắt buộc** của track: "người dùng thuốc ... có dấu hiệu nguy hiểm".

**2. R3 — Không biết người uống là bé 18 tháng**

- **Không đảo được:** người chăm sóc lúc nửa đêm sẽ cho bé uống ngay khi nhận gợi ý; sửa tin nhắn sau đó không lấy lại liều đã uống. Người chịu hại (bé) không phải người đang nhắn và không tự lên tiếng được.
- **Dễ lọt nhất trong 8 kịch bản:** tin nhắn **không chứa một chữ nào báo rủi ro** — không có "bé", "con", "tuổi". Khác R2 (có chữ "bầu"), R5 (có chữ "kháng sinh"), R7 (có chữ "lơ mơ"): những kịch bản đó có từ khoá để chặn. R3 chỉ chặn được nếu thiết kế **luôn hỏi người dùng thuốc trước mọi gợi ý**.
- **Xảy ra thường xuyên nhất:** đúng lời dược sĩ trực trong track ("người sẽ uống là ai thì khách hay quên nói") và lời khách nửa đêm ("con mình sốt... đừng bắt mình chờ"). Ca này không hiếm; nó là ca mỗi đêm.
- Trùng phần **Demo bắt buộc** của track: "người dùng thuốc là trẻ nhỏ".

**Kịch bản suýt được chọn và vì sao không**

- **R2 (ảnh mờ + bà bầu)** — hại cũng không đảo được và là ca "trông đúng nhưng rủi ro". Không chọn vì tin nhắn **tự khai** "đang bầu 12 tuần": một luật cứng "mang thai → không gợi ý, chuyển dược sĩ" chặn được ngay, không cần thiết kế UX phức tạp. R3 thì không có từ khoá nào để bắt.
- **R4 (cả nhà tiêu chảy)** — hại nhân lên 4 người. Không chọn vì khi đã giải được R3 (luôn hỏi người dùng thuốc) và R7 (dấu hiệu nguy hiểm đi trước hàng chờ) thì R4 được xử lý bằng chính hai cơ chế đó, cộng thêm luật "tách yêu cầu". R4 là bài kiểm tra tốt cho thiết kế, không phải một vấn đề thiết kế riêng.
- **R5 (kháng sinh không đơn)** — rủi ro cao nhưng yêu cầu **nói rõ** là ngoài thẩm quyền, nên chặn được ở mức tool: đặt `ban_thuoc_ke_don` = **cấm** và chặn `goi_y_san_pham` khi khách nhắc tới thuốc kê đơn. Không cần thiết kế "khi AI không chắc" vì ở đây AI không có lý do để không chắc.

## §3. Khi AI sai  (chặng 3)
Với mỗi kịch bản đắt nhất:
- AI nói gì (nguyên văn) · người dùng thấy gì · làm gì tiếp · ai được báo, sau bao lâu.
- Quyết định: hỏi lại / trả lời kèm giới hạn / dừng và chuyển người — chọn cách nào, đã loại cách nào, vì sao.
- Loại lỗi (hệ thống / người dùng / ngữ cảnh) và vì sao cách nói, cách sửa đi theo loại đó.
- 2–3 nguyên tắc HAX/PAIR, mỗi cái trỏ vào một chỗ cụ thể (câu chữ hoặc chỗ trên màn hình).
- Lỗi nào sửa câu chữ không cứu được (nếu có), và thiết kế cứu nó thế nào.

## §4. Mức tự chủ  (chặng 4)

Lát cắt có hai hướng sai ngược nhau: **làm liều** (gợi ý thuốc sai, không đảo được: R2, R3, R8) và **bắt chờ** (ca cấp cứu nằm trong hàng chờ: R7). Vì vậy tool *ngăn hại* được **tự làm**, còn tool **gây hại không đảo được** phải qua **người duyệt** hoặc bị **cấm**.

### 4.1 Mức của từng tool

| Tool | Mức | Sai thì ai chịu gì · đảo được không | Chờ người thì mất gì | Phương án đã loại, vì sao |
|---|---|---|---|---|
| `khuyen_di_kham` | **Tự làm**, ngay ở tin đầu khi có cờ đỏ | Báo thừa: khách lo, khách bỏ qua được. Báo thiếu: người bệnh trễ cấp cứu, **không đảo được** (R7). | Đúng những phút quan trọng nhất. Chờ duyệt chính là cái sai của R7. | *Dược sĩ duyệt trước khi hiện*: đêm khuya không ai duyệt kịp thì ca R7 nằm chờ. *Chỉ hiện dòng nhắc nhỏ cạnh gợi ý*: khách vẫn bấm đặt Oresol. |
| `chuyen_duoc_si` | **Tự làm**, kèm thẻ tóm tắt | Chuyển thừa: dược sĩ mất vài phút, **đảo được**. Không chuyển khi cần: trẻ nhỏ, mẹ bầu nhận gợi ý từ AI (R2, R3). | Duyệt việc chuyển *sang người duyệt* chỉ thêm độ trễ. | *Chỉ chuyển khi khách yêu cầu*: khách R3 không biết mình cần dược sĩ. *Hỏi khách trước khi chuyển*: thêm một vòng hỏi cho một việc rẻ và đảo được. |
| `goi_y_san_pham` | **Tách 3 nhánh** (4.2): tự làm / dược sĩ duyệt / cấm | Người dùng thuốc (bé 18 tháng R3, thai nhi R2) chịu hại; **không đảo được** khi thuốc đã dùng. | Thuốc: vài phút, đáng chờ. Vật tư (nước muối, khẩu trang): chờ là vô ích (R1). | Một mức chung cho cả tool: xem 4.2. |
| `luu_ghi_chu_suc_khoe` | **Khách duyệt**, mặc định không lưu | Khách và người nhà bị lộ thông tin sức khoẻ, hoặc bệnh của người nhà bị gắn vào hồ sơ người nhắn (R6). Xoá được, **nhưng lộ thì không thu hồi được**. | Không mất gì: lưu chỉ tiện cho lần sau, không phục vụ ca đang xử lý. | *Tự làm*: R6. *Cấm hẳn*: mất trường hợp lưu có lợi (ghi "dị ứng penicillin" để dược sĩ thấy lần sau). |
| `ban_thuoc_ke_don` | **Cấm** (AI không có nút) | Người dùng thuốc dùng kháng sinh, thuốc ngủ không qua bác sĩ (R5, R6); **không đảo được**. | Không mất gì: kiểm đơn là việc của dược sĩ ở quầy, ngoài luồng AI. | *Khách tự xác nhận "có đơn"*: lời khai không kiểm được. *Dược sĩ duyệt rồi AI bán*: khi đó dược sĩ mới là người quyết. |

**Người duyệt, họ thấy gì, quá hạn thì sao:**
- **`gui_goi_y` nhánh B:** dược sĩ trực duyệt. Dược sĩ thấy thẻ tóm tắt (người dùng thuốc, tuổi, mang thai / cho bú, bệnh nền, thuốc đang dùng, triệu chứng và thời gian, cờ đỏ, nguyên văn tin khách) cùng gợi ý AI soạn, rồi bấm *Gửi* / *Sửa rồi gửi* / *Nhận ca*. Nếu quá 5 phút (giả định), AI **không tự gửi**: báo khách thời gian chờ và cách khác (nhà thuốc gần nhất, hotline), nhắc dấu hiệu cần đi khám, và báo trưởng ca.
- **`luu_ghi_chu_suc_khoe`:** chủ tài khoản duyệt. Họ thấy nguyên văn nội dung sẽ lưu, hồ sơ của ai (bắt buộc chọn), và ai xem được tài khoản này. Không trả lời thì **không lưu**.

### 4.2 Tool khó xếp nhất: `goi_y_san_pham`

| Phương án đã cân nhắc | Lập luận | Cái giá (trỏ về §2) |
|---|---|---|
| **Tự làm** sau sàng lọc | Lát cắt cho AI gợi ý; khách nửa đêm "đừng bắt mình chờ" | **R3**: tin không có chữ nào báo trẻ nhỏ. **R2**: đọc ảnh "trúng" là đúng vì may. Thuốc đã dùng thì không đảo được. |
| **Dược sĩ duyệt mọi gợi ý** | Mọi gợi ý thuốc đều không đảo được | **R1**: mua nước muối cũng phải chờ. Hàng chờ phình to làm **R7, R3 bị chôn** giữa các ca vật tư. |
| **Cấm**, AI chỉ chuyển dược sĩ | An toàn tuyệt đối về phía AI | Như trên, cộng thêm mất giá trị lát cắt; khách bỏ kênh, tự mua theo mạng (R2 xảy ra ngoài hệ thống). |

**Cách chốt.** Ba phương án tranh luận về một tool, nhưng thật ra là ba loại tình huống với ba mức rủi ro. Nhóm **tách tool**: `soan_goi_y` (AI soạn, khách chưa thấy, tự làm) và `gui_goi_y` (gửi kèm nút đặt), rồi đặt mức cho `gui_goi_y` theo **người dùng thuốc và loại sản phẩm**, không theo độ tự tin của model:

| Nhánh | Điều kiện | Mức | Kịch bản |
|---|---|---|---|
| A · Vật tư | Thuộc danh sách "không cần sàng lọc" do dược sĩ phụ trách duyệt trước (nước muối, khẩu trang, nhiệt kế…) | **Tự làm**, khách huỷ được trước khi giao | R1, khẩu trang ở R4 |
| B · Thuốc không kê đơn | Người lớn, đủ 5 trường sàng lọc, không thuộc nhánh C | **Dược sĩ duyệt** | Đau họng người lớn |
| C · Nhóm nguy cơ | Trẻ nhỏ, người ≥ 65 tuổi, mang thai / cho bú, bệnh mạn, đang dùng thuốc kê đơn, tự dùng thuốc nhiều ngày không đỡ, cờ đỏ, hoặc khách nhắc tới thuốc kê đơn | **Cấm**: chỉ `chuyen_duoc_si` (+ `khuyen_di_kham` nếu có cờ đỏ) | R2, R3, R5–R8, phần thuốc ở R4 |

**Luật đi kèm:**
1. **Cổng người dùng thuốc:** thiếu một trong 5 trường (rubric riêng T1 mục 1: ai dùng & tuổi · mang thai / cho bú · thuốc đang dùng · bệnh mạn / dị ứng · bao lâu & đã dùng gì) thì không vào được nhánh B. AI hỏi từng trường, có nút chọn nhanh. Luật này chặn R3 mà không cần từ khoá.
2. **Cờ đỏ đi trước mọi yêu cầu trong cùng tin:** khoá nút đặt và gọi `khuyen_di_kham` ngay (R7).
3. **Tách yêu cầu trong một tin:** vật tư không phải chờ phần thuốc (R4).
4. **Không nói liều; không gợi ý "loại tương tự" cho thuốc kê đơn** (R5; rubric riêng T1 mục 5, 6).
5. **Khách nói "đừng lưu" thì khoá `luu_ghi_chu_suc_khoe` cả phiên** (R6).

### 4.3 Thử mức trên 3 kịch bản

| Kịch bản | AI làm gì | Cái sai bị chặn? | Còn làm khi lẽ ra phải hỏi? | Khách bình thường chờ vô ích? |
|---|---|---|---|---|
| **R7** · Oresol + ông 80 tuổi lơ mơ | Cờ đỏ → `khuyen_di_kham` ở tin đầu ("gọi 115 / đưa ông tới viện ngay"); khoá nút đặt; `chuyen_duoc_si` chạy song song, ưu tiên cao nhất | **Có** | Không hỏi lại "có thật lơ mơ không" (cố ý: báo thừa rẻ hơn hỏi chậm). **Còn lỗi:** mô tả lạ có thể lọt danh sách cờ đỏ, nhưng ông ≥ 65 tuổi vẫn vào nhánh C nên không có gợi ý thuốc. | Không áp dụng |
| **R3** · hạ sốt, không nói ai dùng | Cổng chặn `gui_goi_y` → AI hỏi "Thuốc cho ai dùng?" [Tôi / Trẻ em / Người cao tuổi / Mẹ bầu] → trẻ 18 tháng → nhánh C: chuyển dược sĩ ưu tiên, hỏi cờ đỏ ở trẻ | **Có** | **Còn lỗi:** khách bấm nhầm "Tôi" thì ca vào nhánh B, nhưng dược sĩ duyệt vẫn thấy nguyên văn tin. Đây là lý do nhánh B không được tự làm. | Phải chờ (cố ý), nhưng khách thấy thời gian dự kiến và dấu hiệu cần gọi 115 |
| **R1** · nước muối, 10h tối qua lấy | Nhánh A: trả lời ngay kèm nút đặt giao; nói rõ "chưa kiểm tra được hàng và giờ mở cửa của chi nhánh" | **Có** (chặn bằng câu chữ, vì không có tool tồn kho) | Không, vì sản phẩm không cần sàng lọc | **Không**: không vào hàng chờ dược sĩ |

### 4.4 Lời hứa với người dùng đã đổi

Nếu `goi_y_san_pham` là tự làm, kênh hứa *"Hỏi AI là có thuốc ngay"*. Với 3 nhánh, lời hứa đổi thành:

> *"Trợ lý AI trả lời ngay về sản phẩm thông thường và **báo ngay nếu có dấu hiệu cần đi khám**. Với thuốc, AI hỏi về **người sẽ dùng thuốc**, rồi **dược sĩ xác nhận** trước khi gợi ý tới tay bạn."*

Lời hứa này hiện ở dòng giới thiệu đầu khung chat và trên nhãn "Chờ dược sĩ xác nhận" của thẻ gợi ý thuốc (§5).

## §5. Prototype  (chặng 5)
- Link hoặc file · công cụ · phần nào mock · có lời gọi AI thật không.
- 1–2 ảnh có đánh số; mỗi số: thể hiện gì · nguyên tắc HAX/PAIR nào · mức tự chủ nào.
- Quyết định: cách hiện độ chắc/bằng chứng và điều khiển mức tự chủ — đã nghĩ tới cách nào khác, vì sao chọn cách này.

## §6. Tốt là gì  (chặng 6 — nhẹ, bỏ được nếu thiếu giờ)

**3 tiêu chí Pass/Fail.** Chấm bằng cách đọc hội thoại; người ngoài nhóm chấm cũng ra cùng kết quả.

## §7. Demo
- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.
