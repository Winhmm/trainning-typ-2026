# Caching và Redis trong MySpotify

## 1. Caching

**Cache** là bản sao tạm của dữ liệu hay dùng, đặt ở nơi đọc nhanh hơn database.
VD: `GET /api/genres` không cần SELECT MySQL mỗi lần, đọc từ Redis (RAM) nhanh hơn và giảm tải MySQL.
Đổi lại: dữ liệu có thể **cũ**, code phức tạp hơn.

**Cache ở đâu:**

| | Application-level | Distributed |
|---|---|---|
| Ví dụ | `HashMap`, Caffeine (RAM của app) | Redis |
| Restart app | Mất | Còn |
| Nhiều server | Mỗi server 1 bản, dễ lệch | Dùng chung |

MySpotify chọn **Redis** vì đã có sẵn (đang dùng cho session, rate limit, OTP, chống tăng lượt nghe).

**Khi nào cache:**

| Dữ liệu | Cache? | Lý do |
|---|---|---|
| 25 thể loại | Nên | Đọc nhiều, gần như không đổi |
| Lời bài hát | Nên | Ít đổi |
| Danh sách bài, tìm kiếm | Không | Quá nhiều tổ hợp, bài hẹn giờ xuất hiện theo thời gian |
| Trạng thái tài khoản, `access_status` | Không | Bảo mật, phải có hiệu lực ngay; lại được sửa bằng SQL trực tiếp |

## 2. Chiến lược

- **Cache-Aside:** đọc cache → miss → query DB → ghi cache kèm TTL. Khi dữ liệu đổi: **ghi DB rồi `DEL` key** (không ghi đè, tránh 2 request ghi lệch nhau).
- **Write-Through:** mỗi lần ghi thì ghi cả DB và cache. Đọc luôn nhanh, nhưng ghi chậm hơn và cache cả dữ liệu không ai đọc.

| | Cache-Aside | Write-Through |
|---|---|---|
| Dùng khi | Đọc nhiều, ghi ít (phổ biến nhất) | Ghi xong cần đọc ngay |
| Redis lỗi | Vẫn chạy (đọc DB) | Phải xử lý lỗi khi ghi |
| MySpotify | Thể loại nhạc | Lời bài hát |

## 3. Redis cơ bản

Redis là database **key → value trong RAM**, rất nhanh, key có **TTL** tự hết hạn, lệnh `INCR`/`SET NX` là nguyên tử.

| Kiểu | Ví dụ MySpotify |
|---|---|
| String | `session:{email}`, `rate:user:{email}` |
| Hash | `cache:genres` → `{1: "Pop", 2: "V-Pop"}` |
| List | 10 bài nghe gần nhất (`LPUSH` + `LTRIM`) |
| Set | Id bài đã thích |
| Sorted Set | Bảng xếp hạng lượt nghe (`ZINCRBY`) |

```bash
SET key value EX 60    # ghi, sống 60 giây
GET key
DEL key
EXPIRE key 60
TTL key                # -1: không hạn, -2: không tồn tại

docker run -d --name myspotify-redis -p 6379:6379 redis:7.4
docker exec -it myspotify-redis redis-cli ping   # PONG
```

## 4. Implement (Spring Boot)

Đã có sẵn: `spring-boot-starter-data-redis` (client **Lettuce**), cấu hình `spring.data.redis.host/port`, dùng `StringRedisTemplate`.

Cache thể loại nhạc trong `SongService` (Cache-Aside, Hash, TTL 24 giờ):

```java
public List<GenreResponse> getGenres() {
    try {
        Map<Object, Object> cached = redisTemplate.opsForHash().entries("cache:genres");
        if (!cached.isEmpty()) {
            List<GenreResponse> result = new ArrayList<>();
            cached.forEach((id, name) -> result.add(new GenreResponse(Long.valueOf((String) id), (String) name)));
            result.sort(Comparator.comparing(GenreResponse::getName, String.CASE_INSENSITIVE_ORDER));
            return result;                                   // cache hit
        }
    } catch (Exception e) { }                                // Redis lỗi → đọc DB

    List<GenreResponse> result = genreRepository.findAllByOrderByNameAsc()
            .stream().map(SongMapper::toGenreResponse).toList();   // cache miss

    try {
        Map<String, String> hash = new HashMap<>();
        result.forEach(g -> hash.put(String.valueOf(g.getId()), g.getName()));
        if (!hash.isEmpty()) {
            redisTemplate.opsForHash().putAll("cache:genres", hash);
            redisTemplate.expire("cache:genres",
                    Duration.ofHours(24).plusMinutes(ThreadLocalRandom.current().nextInt(0, 61)));
        }
    } catch (Exception e) { }

    return result;
}
```

Thêm thể loại bằng SQL → chạy `redis-cli DEL cache:genres`.

## 5. Vấn đề thường gặp

| Vấn đề | Là gì | Cách xử lý |
|---|---|---|
| **TTL** | Sống bao lâu | Càng ít đổi càng dài: thể loại 24 giờ, chi tiết bài 30 - 60 giây |
| **Invalidation** | Data đổi mà cache còn cũ | Ghi DB rồi `DEL` key ở **mọi chỗ** sửa dữ liệu; sửa bằng SQL thì `DEL` bằng tay |
| **Avalanche** | Nhiều key hết hạn cùng lúc → DB quá tải | TTL + số ngẫu nhiên (jitter), rate limit |
| **Penetration** | Hỏi data không tồn tại → luôn miss → luôn query DB | Cache giá trị `"__NULL__"` 60 giây, kiểm tra input, rate limit |
