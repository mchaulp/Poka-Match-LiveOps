# Phân vùng & Cấu trúc BXH động 
## Mục lục
1. Tổng quan
2. Các entity và khái niệm
3. Hệ thống phân vùng động
4. Cơ chế đa tâm điểm và chỉ số ưu tiên mô phỏng
5. Quy tắc phân bổ Archetype
6. Lớp thích ứng chế độ vận hành
7. Quy trình vận hành và duy trì
8. Tích hợp với các modules khác
# 1. Tổng quan
## 1.1. Mục tiêu
Tối ưu hóa trải nghiệm cạnh tranh cho từng Real Player (RP) bằng cách đảm bảo rằng:
- Mọi RP luôn cảm thấy mình đang ở trong một môi trường cạnh tranh sống động, công bằng và đầy thử thách.
- Những vị trí trên BXH mà RP quan tâm nhất (gần hạng của họ, top đầu) luôn được dẫn dắt bởi những bot có hành vi chân thực và hấp dẫn nhất.
- Tài nguyên tính toán được phân bổ một cách phù hợp: tập trung vào những khu vực "nóng", nơi có sự chú ý của RP, và giảm thiểu ở những vùng ít được quan tâm.
## 1.2. Nguyên lý
- Lấy người chơi làm trung tâm (Player-Centricity): Mọi phép đo, mọi phân vùng, mọi quyết định về hành vi bot đều được tính toán từ vị trí và góc nhìn của từng RP riêng biệt. Không có khái niệm về một BXH "tĩnh" và "khách quan". BXH là một thực thể (entity) được cá nhân hóa cho từng người chơi.
- Phân vùng động và tương đối (Dynamic & Relative Zoning): Vị trí của các bot không được gán cố định. Chúng liên tục được sắp xếp lại vào các "vùng ảnh hưởng" (DIZ, CPZ, BFZ) dựa trên khoảng cách tương đối với từng RP. Khi RP di chuyển, các vùng này di chuyển theo, đảm bảo trải nghiệm luôn tươi mới và phù hợp.
- Thích ứng linh hoạt với chế độ (Mode Adaptability): Các nguyên tắc cốt lõi là bất biến, nhưng các tham số vận hành (tần suất, tốc độ, phân bổ vai trò) phải được điều chỉnh mạnh mẽ để phù hợp với tính chất của từng loại BXH, dù là mùa giải dài hay sự kiện ngắn.
- Thiết kế cho khả năng mở rộng (Scalability First): Hệ thống được thiết kế để hoạt động hoàn hảo với n=1 RP cũng như n=50 RP. Sự phức tạp tăng lên một cách tự nhiên khi có thêm người chơi thật, nhưng logic nền tảng không hề thay đổi.
## 1.3. Sơ đồ khái niệm

## 1.4. Mối quan hệ với các module khác

# 2. Các entity và khái niệm
Mỗi thực thể được mô tả bằng các thuộc tính (attribute), mối quan hệ và quy tắc tồn tại của nó.

## 2.1. Cohort
Một đơn vị BXH độc lập, chứa 50 slot, là nơi cạnh tranh cho một nhóm Real Player (RP) và Bot (B) trong suốt vòng đời của nó.
| Thuộc tính | Kiểu dữ liệu | Mô tả
| :---: | ---: | ---: |
| `cohort_id` | String | Mã định danh duy nhất của cohort |
| `mode` | Enum (Season, Event) | Chế độ vận hành, quyết định tham số thời gian và hành vi. |
| `slots` | Array[50] | Danh sách 50 vị trí, mỗi vị trí được chiếm bởi một Actor (RP hoặc B). |
| `duration` | Integer | Tổng thời gian tồn tại của cohort. |
| `is_locked` | Boolean | `true` nếu cohort không cho phép thêm/bớt Actor sau khi khởi tạo.  |
- Quy tắc:
	* Mỗi cohort có đúng 50 slot
	* Khi `is_locked = false`, Actor có thể được thêm vào hoặc rời đi sau khi khởi tạo; `is_locked = true`, danh sách Actor là cố định kể từ lúc khởi tạo.

## 2.2. Slot và Actor
- Slot: Một vị trí trên BXH, được xác định bởi hạng (rank) trong khoảng [1, 50].
- Actor: Một thực thể (entity) tổng quát, đại diện cho RP hoặc B đang chiếm giữ một slot. Mỗi Actor có các thuộc tính sau:

| Thuộc tính | Mô tả |
| :---: | ---: |
| `actor_id` | Định danh duy nhất. |
| `actor_type` | RP hoặc B. |
| `current_rank` | Hạng hiện tại, được cập nhật động. |
| `current_score` | Điểm số hiện tại. |
|`join_time`| Thời điểm Actor tham gia cohort. |

## 2.3. Real Player (RP)
- Một người chơi thật, là trung tâm của mọi quyết định trong thiết kế.
- Thuộc tính kế thừa (Inheritance) và mở rộng từ Actor.

| Thuộc tính | Mô tả |
| :---: | ---: |
| `user_id` | ID tài khoản của RP. |
| `skill_proxy` | Giá trị ước lượng kỹ năng của RP. Dùng để khởi tạo Shadow và phân bổ đối thủ phù hợp. Có thể được suy ra từ FTWR trung bình, level hiện tại, hoặc điểm từ mùa trước. |

- Vai trò:
	* Là **trọng tâm** duy nhất để tính toán các vùng ảnh hưởng (DIZ, CPZ).
	* Là **nguồn** cho mọi phép đo sự chú ý, từ đó xác định các chỉ số SP cho Bot.

## 2.4. Bot (B)
- Một thực thể mô phỏng RP, được sinh ra từ Bot Persona, mang trong mình một **Archetype** và các chỉ số hành vi.
- Thuộc tính kế thừa (Inheritance) và mở rộng từ Actor.

| Thuộc tính | Mô tả |
| :---: | ---: |
| `bot_id` | Định danh duy nhất |
| `archetype` | 5 loại: Steady Star; Midnight Rusher; Event Chaser; Social Casual; Shadow. |
| `persona_profile` | Tham chiếu từ Bot Persona. |
| `sp` | Chỉ số ưu tiên mô phỏng hiện tại (1, hoặc 2, hoặc 3); được tính toán động. |
| `assigned_rp` | (Chỉ dành cho Shadow) ID của RP mà Shadow này đang phục vụ; `null` với các Archetype khác. |

## 2.5. Vùng ảnh hưởng (Zones)
Ba vùng đồng tâm, được định nghĩa tương đối cho từng RP, dùng để phân loại mọi slot trên BXH, dựa trên mức độ quan tâm và khả năng chú ý với RP đó.

### 2.5.1. Vùng ảnh hưởng trực tiếp (Direct Influence Zone - DIZ)
- Ký hiệu: $DIZ(RP_i)$
- Định nghĩa: Tập hợp các hạng thỏa mãn một trong hai điều kiện:
	* Thuộc Top 1-5 tuyệt đối, của toàn bộ BXH.
	* Nằm trong khoảng: `[current_rank(RP_i) - 3, current_rank(RP_i) + 3]`
- Công thức:
$DIZ(RP_i) = \{rank \in [1, 50] \mid rank \in [1, 5] \lor |rank - rank(RP_i)| \le 3\}$[^1]
[^1]: Đây là vùng mà RP nhìn thấy ngay lập tức mỗi khi mở BXH, bao gồm cạnh tranh ở top đầu và cạnh tranh ở rank xung quanh họ. Mọi bot trong vùng ngày phải được mô phỏng với độ chân thực cao nhất.

### 2.5.2. Vùng ngoại vi cạnh tranh (Competitive Periphery Zone - CPZ)
- Ký hiệu: $CPZ(RP_i)$
- Định nghĩa: Tập hợp các hạng thỏa mãn đồng thời:
	* Nằm tron khoảng `[current_rank(RP_i) - 15, current_rank(RP_i) + 15]`
	* Không thuộc $DIZ(RP_i)$
- Công thức:
$CPZ(RP_i) = \{rank \in [1, 50] \mid |rank - rank(RP_i)| \le 15 \land rank \notin DIZ(RP_i)\}$[^2]
[^2]: Đây là vùng rộng hơn mà RP có thể vượt qua, trong khả năng của mình, tạo nên bối cảnh cạnh tranh rộng lớn. RP sẽ cuộn xuống để xem họ, nhưng không theo dõi sát sao như ở DIZ.

### 2.5.3. Vùng lấp đầy nền (Background Fill Zone - BFZ)
- Ký hiệu: $BFZ$
- Định nghĩa: Tất cả các rank còn lại không thuộc DIZ hoặc CPZ của bất kỳ RP nào trong cohort.
- Công thức:
$BFZ = \{rank \in [1, 50] \mid \forall RP_i, rank \notin DIZ(RP_i) \land rank \notin CPZ(RP_i)\}$[^3]
[^3]: Đây là phần "chìm" của BXH, nơi RP hiếm khi hoặc không bao giờ nhìn tới. Các bot ở đây chỉ cần tồn tại để lấp đầy 50 slot.

## 2.6. Chỉ số ưu tiên mô phỏng (Simulation Priority - SP)
- Định nghĩa: 
	* Một chỉ số nguyên (1, hoặc 2, hoặc 3) được gán cho mỗi Bot, phản ánh nguồn lực cần thiết trong việc mô phỏng hành vi của Bot đó. 
	* SP được tính toán động dựa trên vị trí của Bot so với tất cả RP trong cohort.
- Công thức:
$SP(B_k) = Max_{i=1..n}(f(B_k, RP_i))$
- Với hàm $f(B_k, RP_i)$, xác định Bot đang nằm ở vùng nào của một RP cụ thể:
	* $f = 3$, nếu $rank(B_k) \in DIZ(RP_i)$
	* $f = 2$, nếu $rank(B_k) \in CPZ(RP_i)$
	* $f = 1$, nếu $rank(B_k) \in BFZ(RP_i)$
- Bảng diễn giải giá trị SP:

| SP | Trạng thái | Ý nghĩa | Hệ quả |
|:---:|---:|---:|---:|
| 3 | Được soi (Scrutinized) | Bot nằm trong vùng quan tâm tối đa của ít nhất một RP | Hành vi được cập nhật với tần suất cao (1-5 phút/lần), Reactivity đầy đủ, mô phỏng chi tiết thắng/thua/dùng booster |
| 2 | Bán soi (Semi-Scrutinized) | Bot nằm trong vùng nhận biết của ít nhất một RP, nhưng không ở vị trí nóng. | Cập nhật với tần suất trung bình (15-30 phút/lần), Reactivity chậm, hành vi ít chi tiết hơn. |
| 1 | Nền (Background) | Bot không nằm trong vùng quan tâm của bất kỳ RP nào. | Cập nhật thưa (1-4 giờ/lần), không Reactivity, chỉ tăng điểm chậm và đều. |

## 2.7.  Chế độ vận hành (Operation Mode)
- Định nghĩa: Một tập hợp các tham số toàn cục điều chỉnh các module vận hành, cho phép cùng một kiến trúc hoạt động linh hoạt cho cả mùa giải dài và sự kiện ngắn.
- Các mode hỗ trợ:

| Mode | Mô tả | Kích hoạt khi |
|:---:|---:|---:|
| `Season` | Mùa giải dài, 7-30 ngày; Cohort động, cho phép RP tham gia muộn và rời đi. | Loại BXH là **Season**. |
| `Event` | Sự kiện ngắn, 24 giờ; Cohort tĩnh, khóa Actor ngay từ đầu. | Loại BXH là **Event**. |

## 2.8. Mối quan hệ giữa các thực thể

# 3. Hệ thống phân vùng động (Dynamic Zoning System)

## 3.1. Mục đích
Giải quyết vấn đồ cốt lõi: Không phải tất cả 50 slot trên BXH đều có giá trị như nhau đối với một RP.
- Họ chỉ thực sự cảm nhận và chịu ảnh hưởng tâm lý từ những vị trí gần họ hoặc những vị trí Top đầu. Những vị trí ở xa chỉ là nền mờ.
- Vì vậy, thay vì phân bổ nguồn lực thiết kế và tính toán một cách dàn trải, cách tối ưu hơn là, định nghĩa ba vùng với mức độ quan trọng giảm dần. Hệ thống sẽ đảm bảo:
	* Tập trung chất lượng vào những khu vực mà RP thực sự quan tâm.
	* Tiết kiệm tài nguyên ở những khu vực ít hoặc không có sự chú ý.
	* Tạo ra nhịp điệu tự nhiên cho BXH: sôi động ở gần, ổn định ở xa.
- Đây là **hệ thống động**, vì các vùng không cố định theo slot, mà liên tục được tính toán lại dựa trên vị trí hiện tại của RP. Khi RP leo hạng, sự chú ý của họ cũng dịch chuyển theo, kéo theo sự thay đổi vai trò của các bot xung quanh. 
- Điều này phải đúng trong cả Season Mode và Event Mode, để đảm bảo trải nghiệm luôn chân thực ngay cả khi RP thay đổi thứ hạng nhanh chóng trong thời gian ngắn.

## 3.2. Thuật toán

- Cho một $RP_i$ với rank hiện tại là $R_i =$ `current_rank(RP_i)`. 
- Các vùng $DIZ(RP_i)$ và $CPZ(RP_i)$ được xác định như các tập hợp số nguyên (từ 1 đến 50) theo các bước sau.

### 3.2.1. Xác định DIZ
- Mục đích: Nắm bắt mọi slot mà $RP_i$ có khả năng nhìn thấy ngay lập tức khi mở BXH, hoặc những vị trí mang tính biểu tượng (Top đầu) mà bất kỳ RP nào cũng nhìn qua.
- Định nghĩa:
$DIZ(RP_i) = \{rank \in [1, 50] \mid rank \in [1, 5] \lor |rank - R_i| \le 3\}$
- Các bước tính:
	1. Khởi tạo tập rỗng `DIZ_set`.
	2. Thêm các rank từ 1 đến 5 vào `DIZ_set`.
	3. Tính $lower = Max(1, R_i - 3)$ và $upper = Min(50, R_i + 3)$.
	4. Thêm tất cả các rank trong $[lower, upper]$ vào `DIZ_set`.
	5. Loại bỏ trùng lặp nếu có.
- Lưu ý: Top 5 luôn nằm trong DIZ của mọi RP, bất kể RP đang ở rank bao nhiêu. Lý do: ngay cả những RP yếu nhất cũng quan tâm về *người dẫn dầu*. Điều này đảm bảo cuộc đua Top đầu luôn được mô phỏng chất lượng cao và hấp dẫn cho tất cả mọi RP trong cohort.

### 3.2.2. Xác định CPZ
- Mục đích: Nắm bắt những đối thủ mà $RP_i$ có thể chạm tới nếu họ cố gắng cạnh tranh, tạo nên bối cảnh cạnh tranh rộng hơn và cảm giác về một cộng đồng đông đúc.
- Định nghĩa:
$CPZ(RP_i) = \{rank \in [1, 50] \mid |rank - R_i| \le 15 \land rank \notin DIZ(RP_i)\}$[^4]
[^4]: Ngưỡng $\pm15$ bậc, được chọn dựa trên ước tính khả năng leo hạng trung bình của một RP bình thường. Con số này có thể được tinh chỉnh sau khi có dữ liệu thật, nhưng nó tạo ra một vùng đệm đủ rộng để bao phủ những đối thủ tiềm năng mà RP có thể lên kế hoạch vượt qua.
- Các bước tính:
	1. Khởi tạo tập rỗng `CPZ_set`.
	2. Tính $lower = Max(1, R_i - 15)$ và $upper = Min(50, R_i + 15)$.
	3. Với mỗi rank trong $[lower, upper]$, nếu rank đó không có trong `DIZ_set` thì thêm vào `CPZ_set`.

### 3.2.3. Xác định BFZ
- Mục đích: Tập hợp tất cả các slot còn lại, nơi mà $RP_i$ không bao giờ hoặc rất hiếm khi chú ý đến.
- Định nghĩa:
$BFZ(RP_i) = \{rank \in [1, 50] \mid rank \notin DIZ(RP_i) \land rank \notin CPZ(RP_i)\}$.
- Các bước tính: Lấy **phần bù** của hợp $DIZ(RP_i) \cup CPZ(RP_i)$ trong toàn bộ 50 slot.
- Đặc điểm: Trong trường hợp có nhiều RP, BFZ toàn cục (dùng cho cả cohort) là phần giao của tất cả BFZ của từng RP, tức là, những slot không nằm trong vùng quan tâm của bất kỳ ai.

## 3.3. Cập nhật vùng khi RP di chuyển

### 3.3.1. Nguyên tắc chung

Vị trí của RP thay đổi liên tục khi họ cày điểm. Mỗi khi $RP_i$ tăng hoặc giảm rank, hệ thống phải tính toán lại $DIZ(RP_i)$ và $CPZ(RP_i)$. Quy tắc này, phải áp dụng thống nhất cho cả Season Mode và Event Mode.
- Tuần suất cập nhật:
	* Ngay lập tức, sau mỗi lần điểm số của $RP_i$ thay đổi đủ để thay đổi rank.
	* Trong các Event nước rút hoặc giai đoạn cuối mùa, có thể bổ sung cập nhật định kỳ mỗi 1-2 phút để đảm bảo tính kịp thời ngay cả khi RP chưa đổi rank nhưng điểm số thay đổi nhanh.
- Quy trình:
	1. Bắt `event` "$RP_i$ thay đổi rank".
	2. Xác định lại $R_i$ mới.
	3. Tính lại $DIZ(RP_i)$, theo 3.2.1.
	4. Tính lại $CPZ(RP_i)$ theo 3.2.2.
	5. BFZ tự động thay đổi theo.
	6. Kích hoạt quy tình tính lại SP cho tất cả các Bot bị ảnh hưởng.
- Hệ quả:
	* Khi RP thăng hạng, một số Bot từ CPZ có thể rơi vào DIZ mới, và ngược lại. Điều này, tạo ra sự **thay đổi vai trò** liên tục cho Bot, giống như RP khi đến gần một đối thủ thì sẽ bắt đầu chú ý đến đối thủ đó hơn.
	* Một Bot bị RP vượt qua sẽ từ DIZ tụt xuống CPZ, mức độ mô phỏng giảm. Điều này phản ánh thực tế: Khi RP bỏ xa một đối thủ, thì họ sẽ không còn quan tâm nhiều đến đối thủ đó nữa.
