# Bài nộp LAB5 — T1 · Nhóm E3
**Lớp:** L3B · **Bàn / zone:** E3 · **Track:** T1

| Họ và Tên | Mã Học Viên | Việc đã làm |
| Nguyễn Đức Minh | 2A202603765 |---|
| Phạm Thế Hùng |  |  |
| Đào Thị Ngọc Ánh |  |  |
|  |  |  |


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
Làm cho hai kịch bản đắt nhất ở §2: **R7** (cấp cứu lẫn trong câu mua hàng) và **R3** (không biết người uống là bé 18 tháng). Các con số thời gian bên dưới (2 phút, 3 phút) là **nhóm tự đặt để thiết kế**; track không có dữ liệu thời gian chờ dược sĩ thật.

### 3.1 · R7 — "Đặt 2 hộp Oresol giao gấp… ông nội 80 tuổi… cứ lơ mơ"

**Quyết định: dừng việc bán, khuyên đi cấp cứu ngay, báo dược sĩ song song.**

| Phương án | Chọn / loại | Vì sao |
|---|---|---|
| **Dừng và khuyên đi khám ngay** (`khuyen_di_kham` ở tin trả lời đầu tiên, không hiện nút đặt) | **Chọn** | Ở R7 cái hại là *phút đã mất*. Khuyên thừa thì khách bỏ qua được (track: "Khách bỏ qua được"), mất vài giây. Khuyên thiếu thì không lấy lại được. Hai phía lệch giá quá xa nên không cần chắc mới nói. |
| Hỏi lại một câu ("Ông có trả lời được khi gọi không?") | Loại | Khách đã tả đủ dấu hiệu ("gọi ông không tỉnh lắm, cứ lơ mơ"). Hỏi thêm là lấy thời gian của ca cấp cứu để mua sự chắc chắn cho AI. |
| Trả lời kèm giới hạn (đặt Oresol, kèm dòng "nếu ông lơ mơ nên đi khám") | Loại | Nút "Đặt ngay" là lời mời *chờ thuốc giao tới*. Người nhà đang rối sẽ bấm nút, không đọc dòng cảnh báo. |
| Chỉ chuyển dược sĩ (`chuyen_duoc_si`) | Loại | Trông an toàn nhưng đưa ca vào hàng chờ, đúng điều lát cắt cấm ("ca nguy hiểm không nằm trong hàng chờ"). Dược sĩ vẫn được báo, nhưng khách **không phải chờ** dược sĩ. |

**AI nói gì (nguyên văn):**

> **Ông đang lơ mơ, gọi không tỉnh hẳn sau khi nôn cả ngày. Đây là dấu hiệu cần cấp cứu, không chờ thuốc giao tới được.**
> **Bạn gọi 115 hoặc đưa ông tới bệnh viện gần nhất ngay bây giờ.**
>
> Mình là trợ lý AI, không khám được cho ông, nên có thể mình đang lo quá mức. Nhưng với dấu hiệu này, đi viện thừa vẫn an toàn hơn ngồi chờ.
> Mình **chưa đặt** Oresol. Bạn không cần chờ dược sĩ trả lời rồi mới đi.

| | Thiết kế |
|---|---|
| **Người dùng thấy gì** | (a) Thẻ đỏ ghim trên đầu khung chat, không trôi theo tin nhắn. (b) Nút lớn **"Gọi 115"** (bấm là mở cuộc gọi). (c) Dòng **"Vì sao mình nói vậy"**: trích nguyên văn chữ của khách — *"80 tuổi" · "nôn ói từ sáng" · "gọi ông không tỉnh lắm, cứ lơ mơ"*. (d) Dòng trạng thái đơn: **"Oresol: chưa đặt"**; không có nút đặt. (e) Dòng trạng thái do hệ thống sinh: **"Đã báo dược sĩ trực lúc 21:04"**, chỉ hiện khi `chuyen_duoc_si` chạy xong. (f) Nút phụ **"Ông vẫn tỉnh táo, mình tả chưa đúng"**. |
| **Làm gì tiếp** | Bấm "Gọi 115" → thẻ vẫn ghim. Bấm "Ông vẫn tỉnh táo…" → AI **không** tự gỡ cảnh báo để quay lại bán: thẻ thu nhỏ nhưng còn đó, ca được đẩy lên đầu hàng dược sĩ, và chỉ dược sĩ mới mở lại được việc đặt Oresol. Khách không làm gì trong 2 phút → AI nhắn **một** lần: "Bạn đã gọi 115 hoặc đưa ông đi chưa?" rồi thôi, không nhắn dồn. |
| **Ai được báo, sau bao lâu** | **Dược sĩ trực, ngay lập tức** (song song với tin trả lời khách, không chờ khách bấm gì). Dược sĩ thấy thẻ đỏ ở đầu hàng: nguyên văn tin khách, ba dấu hiệu AI bắt được, việc AI đã làm (đã hiện khuyên cấp cứu lúc mấy giờ, chưa đặt Oresol), và khách đã bấm nút nào. Quá 3 phút chưa dược sĩ nào mở → hệ thống báo lại và báo trưởng ca. |

**Loại lỗi và cách nói, cách sửa:**
- **AI bán Oresol như một đơn bình thường = lỗi ngữ cảnh.** Hệ thống chạy đúng thiết kế "xử lý yêu cầu mua hàng" nhưng sai với tình huống. Sửa câu xin lỗi không có ích; phải sửa thiết kế: bước kiểm dấu hiệu nguy hiểm chạy trên **cả tin nhắn** và chạy **trước** mọi lần gọi `goi_y_san_pham`.
- **AI báo cấp cứu mà ông chỉ đang buồn ngủ = lỗi hệ thống** (AI đánh giá quá mức). AI nhận phần mình ngay trong câu đầu ("có thể mình đang lo quá mức"), không bắt khách chứng minh, và đường sửa là nút (f) dẫn tới người thật.
- **Khách tự quyết "chỉ cần Oresol"** không được coi là lỗi người dùng: câu chữ không trách khách đặt sai, chỉ nói điều cần làm ngay.

### 3.2 · R3 — "Shop ơi có thuốc gì hạ sốt nhanh ko, sốt 39 độ từ chiều…"

**Quyết định: hỏi lại đúng một câu "thuốc này cho ai dùng" trước mọi gợi ý.**

| Phương án | Chọn / loại | Vì sao |
|---|---|---|
| **Hỏi lại một câu, có nút chọn nhanh** | **Chọn** | Thứ còn thiếu chỉ là một dữ kiện, khách trả lời bằng một lần bấm (vài giây). Đổi lại, chặn được cái sai không đảo được: gợi ý viên người lớn cho bé 18 tháng. |
| Trả lời kèm giới hạn ("Nếu là người lớn thì dùng loại này…", kèm nút đặt) | Loại | Người chăm bé lúc nửa đêm đọc tên thuốc và bấm nút, bỏ qua chữ "nếu". Giới hạn viết bằng chữ không cản được nút bấm. |
| Dừng và chuyển dược sĩ mọi câu hỏi hạ sốt | Loại | Người lớn sốt thông thường cũng phải chờ, trái lời khách trong track ("đừng bắt mình chờ"). Hàng chờ đầy ca nhẹ thì ca nặng như R7 chờ lâu hơn. Dược sĩ rồi cũng phải hỏi đúng câu đó. |
| Đoán từ ngữ cảnh (không có chữ "bé" → người lớn) | Loại | Không có căn cứ nào trong tin nhắn; đúng lỗi "mặc định người nhắn là người uống" ở §2. |

**AI nói gì (nguyên văn) — lượt 1, khi chưa biết ai dùng thuốc:**

> Mình giúp ngay. Thuốc hạ sốt cho trẻ nhỏ, người lớn và mẹ bầu khác nhau, nên mình cần biết một điều trước khi gợi ý:
> **Thuốc này cho ai dùng ạ?**
>
> `Mình (người lớn)` · `Trẻ dưới 2 tuổi` · `Trẻ 2–12 tuổi` · `Người trên 65 tuổi` · `Đang mang thai / cho con bú`

**Lượt 2, sau khi khách bấm "Trẻ dưới 2 tuổi" và nhắn "bé 18 tháng":**

> Bé 18 tháng sốt 39 độ: mình là trợ lý AI nên **không tự gợi ý thuốc hay liều cho bé**. Mình đã chuyển ca cho dược sĩ kèm tóm tắt, bạn không phải kể lại. Mình chưa biết chắc bao lâu dược sĩ trả lời.
>
> Trong lúc chờ, bé có dấu hiệu nào dưới đây không? Có một dấu hiệu là cần đưa bé đi viện ngay, không chờ dược sĩ.
>
> `Co giật` · `Li bì, khó đánh thức` · `Khó thở` · `Bỏ bú, không uống được` · `Không có dấu hiệu nào`

| | Thiết kế |
|---|---|
| **Người dùng thấy gì** | (a) Nhãn **"Trợ lý AI"** cạnh tên người gửi ở mọi tin. (b) Hàng nút chọn nhanh thay cho ô gõ; chưa bấm thì **không có thẻ sản phẩm, không có nút đặt nào** trên màn hình. (c) Thẻ tóm tắt gửi dược sĩ, khách xem được: *"Người dùng thuốc: bé 18 tháng · sốt 39°C từ chiều · chưa rõ cân nặng, thuốc đã dùng"*. (d) Dòng trạng thái **"Đã chuyển dược sĩ lúc 23:42 · chưa có giờ trả lời chắc chắn"**. (e) Với ca người lớn được gợi ý: trên thẻ sản phẩm luôn có **"Gợi ý này cho: Người lớn · Đổi"**. |
| **Làm gì tiếp** | Bấm một dấu hiệu nguy hiểm → chuyển ngay sang thẻ đỏ như R7 (`khuyen_di_kham`, nút "Gọi 115"). Bấm "Không có dấu hiệu nào" → chờ dược sĩ; danh sách dấu hiệu vẫn nằm trên màn hình. Khách **không trả lời "cho ai"** hoặc gắt ("nói luôn đi") → AI không mặc định người lớn: nói một câu lý do, giữ nguyên câu hỏi, và sau 2 phút im lặng thì chuyển dược sĩ với nhãn *"chưa rõ người dùng thuốc"*. |
| **Ai được báo, sau bao lâu** | **Dược sĩ trực, ngay khi biết người dùng thuốc là trẻ dưới 2 tuổi** (không chờ khách trả lời câu dấu hiệu nguy hiểm). Ca vào hàng ưu tiên, dược sĩ thấy thẻ tóm tắt (c) và câu trả lời dấu hiệu khi khách bấm. Quá 3 phút chưa ai mở → báo lại, và AI nói thật với khách: "Dược sĩ chưa nhận ca. Nếu bạn thấy bé nặng hơn, đưa bé đi viện, đừng chờ." |

**Đường sửa (correction) — khi AI đã lỡ gợi ý rồi khách mới nói "ủa cho bé 18 tháng mà":**

> Mình đã sai: mình gợi ý khi chưa biết đúng người dùng thuốc. Gợi ý viên nén vừa rồi **không dùng cho bé**, bạn đừng cho bé uống.
> Bé đã uống chưa?
>
> `Chưa uống` · `Đã uống rồi`

Thẻ sản phẩm cũ bị gạch và nút đặt bị tắt ngay. "Đã uống rồi" → chuyển dược sĩ ưu tiên kèm tên sản phẩm đã gợi ý, đồng thời hiện danh sách dấu hiệu phải đi viện. "Chưa uống" → quay lại luồng lượt 2.

**Loại lỗi và cách nói, cách sửa:**
- **AI mặc định người nhắn là người uống = lỗi ngữ cảnh.** Khách không sai khi viết thiếu chủ ngữ; người thật nhắn như vậy. Vì thế câu chữ không nói "bạn chưa cung cấp đủ thông tin" mà nói "mình cần biết một điều". Cách sửa là thiết kế: `goi_y_san_pham` không gọi được khi ô "người dùng thuốc" còn trống.
- **Khách bấm nhầm nút (chọn "Mình" trong khi mua cho con) = lỗi người dùng.** Không cần xin lỗi dài, chỉ cần sửa nhanh: nút **"Đổi"** ở (e) nằm ngay trên thẻ gợi ý.
- **AI đã gợi ý sai rồi mới biết = lỗi hệ thống.** AI nhận lỗi bằng chữ "mình đã sai", nói rõ sai ở đâu, và việc đầu tiên là chặn hại ("đừng cho bé uống"), sau đó mới giải thích.

### 3.3 · Nguyên tắc HAX/PAIR và chỗ cụ thể

- **G10 — Thu hẹp phạm vi khi nghi ngờ** → R3 lượt 1: câu "Thuốc này cho ai dùng ạ?" kèm 5 nút, thay cho thẻ sản phẩm. Khi chưa biết người dùng thuốc, AI chỉ làm một việc là hỏi.
- **G11 — Giải thích vì sao** → R7 (c): dòng "Vì sao mình nói vậy" trích ba cụm chữ của chính khách. R3 lượt 1: nửa câu "thuốc hạ sốt cho trẻ nhỏ, người lớn và mẹ bầu khác nhau" đứng *trước* câu hỏi, để khách đang vội hiểu vì sao bị hỏi lại.
- **G9 — Sửa dễ dàng** → R3 (e): "Gợi ý này cho: Người lớn · Đổi". R7 (f): nút "Ông vẫn tỉnh táo, mình tả chưa đúng".
- **G1/G2 — Làm rõ hệ thống làm được gì, tốt đến đâu** → nhãn "Trợ lý AI" ở R3 (a); câu "Mình là trợ lý AI, không khám được cho ông, nên có thể mình đang lo quá mức" ở R7.
- **PAIR · Errors + Graceful Failure** → mỗi loại lỗi một đường lui riêng: nút "Đổi" cho lỗi người dùng, luồng "Bé đã uống chưa?" cho lỗi hệ thống, nút (f) dẫn tới dược sĩ cho báo động thừa.

### 3.4 · Lỗi sửa câu chữ không cứu được

1. **Nút "Đặt ngay" đứng cạnh lời cảnh báo.** Ở cả R7 và R3, dù câu cảnh báo viết hay tới đâu, chừng nào còn nút đặt trên màn hình thì người đang vội vẫn bấm. → Thiết kế cứu: nút đặt **không tồn tại** trong hai trạng thái đó. `goi_y_san_pham` bị khoá khi (i) tin nhắn có dấu hiệu nguy hiểm, hoặc (ii) ô "người dùng thuốc" còn trống hoặc thuộc nhóm nguy cơ. Đây là luật sẽ ghi ở §4.
2. **AI nói "mình đã báo dược sĩ" khi chưa có ai được báo** (tool lỗi, hoặc không có dược sĩ trực lúc 0h — track không có dữ liệu giờ trực). Sửa câu chữ cho khiêm tốn hơn vẫn là một lời hứa AI không kiểm được. → Thiết kế cứu: câu đó **không nằm trong lời AI**. Nó là dòng trạng thái do hệ thống sinh từ kết quả `chuyen_duoc_si` (R7 (e), R3 (d)); tool lỗi thì dòng đó hiện "Chưa chuyển được cho dược sĩ" và thẻ khuyên đi viện vẫn đứng độc lập.

## §4. Mức tự chủ  (chặng 4)
Với mỗi tool:
- Mức của nhóm (tự làm / cần người duyệt / cấm). Nếu cần người duyệt: ai duyệt · người duyệt thấy gì · quá hạn thì sao.
- Lý do theo cost-of-error: sai thì ai chịu gì · đảo lại được không · chờ người thì mất gì.
- Phương án đã cân nhắc và vì sao loại.
Thêm:
- Tool khó xếp nhất: mỗi người từng chọn gì, cái giá của mỗi phía (trỏ về kịch bản ở §2), nhóm chốt thế nào.
- Thử mức đã đặt trên 2 kịch bản đắt nhất + 1 kịch bản bình thường: cái sai có bị chặn không? còn việc nào làm khi lẽ ra phải hỏi? người dùng bình thường có phải chờ vô ích không?
- Luật thêm hoặc tool tách bước (nếu có), và vì sao.
- Một mức tự chủ đã đổi lời hứa nào với người dùng.

## §5. Prototype  (chặng 5)
- Link hoặc file · công cụ · phần nào mock · có lời gọi AI thật không.
- 1–2 ảnh có đánh số; mỗi số: thể hiện gì · nguyên tắc HAX/PAIR nào · mức tự chủ nào.
- Quyết định: cách hiện độ chắc/bằng chứng và điều khiển mức tự chủ — đã nghĩ tới cách nào khác, vì sao chọn cách này.

## §6. Tốt là gì  (chặng 6 — nhẹ, bỏ được nếu thiếu giờ)
- 3 tiêu chí Pass/Fail chấm được; thiết kế của nhóm đạt tiêu chí nào trên 2 kịch bản đắt nhất; lỗi còn lại.

## §7. Demo
- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.
