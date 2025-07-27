# 🐧 Linux Learning | Phát triển Linux nhúng

## 📚 Static Library vs Shared Library | So sánh thư viện tĩnh và động

| Properties | Static Library | Shared Library |
|------------|---------------|----------------|
| Linking time | Tất cả các modules trong thư viện sẽ được copy vào trong file thực thi tại thời điểm biên dịch (compile time). Khi chương trình được load vào bộ nhớ, OS chỉ đặt một file thực thi duy nhất bao gồm source code và thư viện được link (static linking) | Trong khi đó, shared lib được sử dụng trong quá trình link khi mà cả file thực thi và file thư viện đều được load vào bộ nhớ (runtime). Một shared lib có thể được nhiều chương trình sử dụng. (Dynamic linking) |
| Size | Sử dụng static lib tốn nhiều bộ nhớ hơn shared lib | Sử dụng shared lib tốn ít bộ nhớ hơn vì chỉ có duy nhất một bản sao trong bộ nhớ |
| External File changes | File thực thi cần phải recompile lại bất cứ khi nào có sự thay đổi trong static lib | Đối với shared lib, không cần phải biên dịch lại file thực thi |
| Time | Mất nhiều thời gian hơn để thực thi | Sử dụng shared lib tốn ít thời gian hơn để thực thi vì thư viện nằm sẵn trong bộ nhớ |