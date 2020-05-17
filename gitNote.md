## I. Cấu hình git
- Trước khi làm việc với git phải tiến hành cấu hình. Mọi dữ liệu được bạn tạo ra ở git sẽ ăn theo thông tin cấu hình này.

- Chú ý :không cấu hình sẽ không chạy được 1 số câu lệnh như:
  -  commit
  -  clone
  -  stash
  
**1. Lệnh cấu hình Tên và Email**
```js
git config --global user.name "Tên của bạn"
git config --global user.email "Email của bạn"
```
**2. Kiểm tra lại config xem đã đúng chưa**
```js
git config --list
```

## II. Repository
- Repository là gì: Là kho lưu trữ dữ liệu của 1 dự án.
  
- Local Repository: Là repo phía local của developer, nó được đồng bộ lên remote repo khi bạn làm việc với git.
  
- Remote Repository: Là repo được lưu trữ ở git trung tâm ví dụ như github, bitbucket, backlog

**Lệnh tạo repo:**
```js
mkdir [tên dự án]
cd [tên dự án]
git init
```

## III. Git Branch
- Branch là gì: Trong 1 dự án có nhiều task vụ khác nhau, chính vì vậy người ta phải dùng branch để phân nhánh mỗi task vụ đó thành 1 branch riêng biệt để lưu lại luồng lịch sử làm việc cho toàn bộ dự án. Branch đã phân nhánh sẽ không ảnh hưởng đến branch khác nên có thể tiến hành nhiều thay đổi đồng thời trong cùng 1 repository

1. Branch Master
2. Branch Release(Staging)
3. Branch Develop
4. Branch Feature
5. Branch Hostfix

## IV. Các lệnh liên quan tới branch
- Khi 1 repo mới được tạo ra thì nó sẽ chưa tồn tại bất kỳ nhánh nào, để khởi tạo 1 branch thì ta phải lưu trữ trong branch ít nhất 1 file thì nó sẽ mặc định tạo ra branch master đầu tiên

Ví dụ:
- Step 1: Tạo file hello.txt
```js
 echo “Hello Tin Hoc That La Don gian”　＞　hello.txt
```
- Step2: Chạy lệnh git add để thêm file vào git history
```js
 git add hello.txt
```
- Step3: 
```js
 git commit -m “Dua file hello.txt vao nhanh”
```

#### 1. Lệnh xem các branch đang có trong repo
```js
git branch
```
#### 2. Lệnh tạo branch
- Tạo branch mới và đứng nguyên ở branch hiện tại
```js
git branch [tên branch]
```

**chú ý**: khi bạn đang đứng trong 1 branch để tạo 1 nhánh mới thì, nhánh mới được tạo ra sẽ copy toàn bộ dữ liệu của nhánh hiện tại qua 1 nhánh mới

- Tạo branch mới và chuyển luôn qua branch mới
```js
git checkout -b [tên branch mới]
```

#### 3. Lệnh xem trạng thái thay đổi của 1 branch
```js
git status
```
#### 4. Lệnh chuyển đổi giữa các branch
```js
git checkout [tên branch]
```

 **Chú ý**: trước khi chuyển branch phải commit code lên branch hiện tại không thì code sẽ bị mất

#### 5. Lệnh đưa dữ liệu lên 1 branch: git add và git commit
- Trước khi đưa dữ liệu lên 1 branch bạn phải dùng lệnh git add để khai báo bạn muốn đưa những gì lên nhánh

- Đưa từng file
```js
git add [file 1] [file 2] [thư mục 1/*]
```
- Đưa toàn bộ những thanh đổi lên branch
```js
git add .
git commit -m “Comment nội dung mà bạn đưa lên brach”
```

#### 6. Lệnh Git merge branch
```js
git merge ＜tên nhánh＞
```
- Ví dụ có 2 nhánh A và B tiến hành merge B vào A
Step 1: git checkout [nhánh A]
Step 2: git merge B

##### 6.1 Xử lý conflict(xung đột) khi merge code
- Đoạn bị xung đột được bắt đầu bằng 
```js
＜＜＜＜＜＜＜ HEAD 
  
Và kết thúc tại 

＞＞＞＞＞＞＞ 
```

- Sửa conflict xong thì tiến hành đưa lại nội dung lên branch được merge vào
```js
Step 1: Tiến hành sửa conflict
Step 2: Sau khi sửa thì tiến hành đưa lại code muốn merge lên nhánh muốn merge

git add .
git commit -m “Sửa conflict”
```
#### 7. Xóa branch
- Khi toàn bộ dữ liệu trên branch đó đã được merge vào master thì dùng lệnh sau để xóa
```js
git branch -d ＜tên branch cần xóa＞
```
**Chú ý :** với câu lệnh trên thi bắt buộc dữ liệu thì branch muốn xóa phải được merge vào master, không thì bạn sẽ nhận được báo lỗi.

- Trường hợp branch muốn xóa chưa được merge vào master thì bạn phải dùng câu lệnh
```js
 git branch -D ＜tên branch cần xóa＞
```
**Chú ý :** trường hợp này toàn bộ các commit lên branch này sẽ bị mất

#### 8. Lệnh tra lại lịch sử
```js
git log
```

#### 9. Thay đổi nội dung commit lần trước

- Sửa lại message commit bị nhầm
```js
git commit --amend

git push -f
```

- Trường hợp add thiếu file hoặc sửa lại file đã commit
```js
git add [file bị thiếu hoặc file cần sửa lại]
git commit --amend

git push -f
```

#### 10. Undo lại commit
- Trường hợp 1: Muốn "undo" thay đổi trên một file cố định **trước khi commit**
```js
git checkout -- [đường dẫn/tên file]
```

- Nếu muốn undo hẳn một commit (do đã lỡ commit xong rồi) thì cần chỉ định mục tiêu

```js
git reset --soft HEAD~1

Ở đây HEAD~1 nghĩa là trước 1 commit. Mình dùng soft để lưu 
lại những thay đổi chưa commit và chỉ bỏ đi phần đã commit từ
lần trước
```

- Nếu muốn bỏ cả phần đã commit từ lần trước và phần chưa commit thì đổi soft thành hard
```js
git reset --hard HEAD~1
```

#### 11. Xóa hết các files chưa được commit
```js
git clean --force
```

#### 12. Thay đổi tên tác giả của commit
```js
git config user.name "Nguyen Van Doanh"
git config user.email doanh@gmail.com
git commit --amend --reset-author
```
#### 13. Đưa code từ local repo lên remote repo
- 1.Tạo tài khoản github
  
- 2.Tạo repository trên github
  
- 3.Thêm remote repository vào local repository
```js
- Thêm vào
git remote add origin [git remote url]

- Check lại xem thêm vào ok hay chưa
git remote -v

- Thay đổi link
git remote set-url origin [git remote url]
```

- 4. Đưa code từ local repo lên remote repo
```js
git add .
git commit -m “commit comment”
git push origin [tên remote branch]
```
