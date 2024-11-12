# CTF writeup
Thử thách này rất hay cũng đã mất của mình rất nhiều thời gian và mình đã build lại để giải thử trên localhost.
Cụ thể giao diện khi các bạn truy cập vào web thử thách sẽ là :
![image](https://github.com/user-attachments/assets/8b4f4651-d0f0-43a1-a89b-6a25660d798a)

    <?php
    
    function fil($str)
    {
        return str_replace("BlueCyber", "BC", $str);
    }
    
    class x
    {
        public $username;
        public $password;
        public $isAdmin = false;
    
        public function __construct($username, $password)
        {
            $this->username = $username;
            $this->password = $password;
        }
    
        public function __wakeup()
        {
            if ($this->isAdmin) {
                // Ensure the file inclusion is controlled and safe
                if (file_exists("flag.php")) {
                    include "flag.php";
                    echo 'Awesome! Here is your flag: ' . htmlspecialchars($flag, ENT_QUOTES, 'UTF-8');
                } else {
                    echo 'Flag file not found.';
                }
            } else {
                echo 'Incorrect credentials.<br>';
            }
        }
    }
    
    $username = isset($_GET['username']) ? $_GET['username'] : '';
    $password = isset($_GET['password']) ? $_GET['password'] : '';
    
    $ser = fil(serialize(new x($username, $password)));
    $o = @unserialize($ser);
    
    if (isset($_GET['debug'])) {
        highlight_file(__FILE__);
    }
    ?>
    
đây là code php của chương trình này, chúng ta sẽ quan tâm đến hàm wakeup để lấy cờ hãy cùng xem điều kiện như thế nào sẽ đủ để lấy cờ:

    public function __wakeup()
        {
            if ($this->isAdmin) {
                // Ensure the file inclusion is controlled and safe
                if (file_exists("flag.php")) {
                    include "flag.php";
                    echo 'Awesome! Here is your flag: ' . htmlspecialchars($flag, ENT_QUOTES, 'UTF-8');
                } else {
                    echo 'Flag file not found.';
                }
            } else {
                echo 'Incorrect credentials.<br>';
            }
        }
Chúng ta thấy chỉ cần this->admin thì nó sẽ kiểm tra file flag có tồn tại hay ko và nó sẽ in ra cho chúng ta. Nhưng ở đây ta phải hiểu là this ở đây đại diện cho đối tượng của class x. Class x này được định nghĩa là đối tượng có 3 tham số username, password và isAdmin và ta chỉ có 2 đầu vào có thể thay đổi là username, password

    $username = isset($_GET['username']) ? $_GET['username'] : '';
    $password = isset($_GET['password']) ? $_GET['password'] : '';
    
    $ser = fil(serialize(new x($username, $password)));
    $o = @unserialize($ser);
ở đây ta thấy nó lấy 2 tham số này với phương thức get xong rồi sẽ như vào hàm serilize. Hàm serilize là hàm mà 
khi một đối tượng thì nó mã hóa cho bạn theo một chuỗi có logic . Bạn có thể xem ở đây: https://www.php.net/manual/en/function.serialize.php
và hàm unserizlize có nghĩa là nó sẽ giải mã ngược lại từ hàm serilize trên.
ở đây giả sử ta điền username=1 và password=1 thì hàm serialize nó sẽ là : ![image](https://github.com/user-attachments/assets/8a626b09-79dd-4e6e-8f5a-d1f7be6f9974)
lợi dụng chỗ này vì unserilize sẽ lấy nội dung từ trái sang phải để hiểu và giải mã các phương thức của đối tượng nên ta sẽ có payload như sau :
![image](https://github.com/user-attachments/assets/9a0bc92f-6ab5-4aa9-bbac-63ae9260da94)
ta sẽ lợi dụng hàm fil thay thế từ ở đây và chèn chuỗi bypass.


    
        
