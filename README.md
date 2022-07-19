# php-ques

### Algorithm
    Brush more leetcode
    
#### 1. Give you four coordinate points, judge whether they can form a rectangle, such as judge ([0,0],[0,1],[1,1],[1,0]) can form a rectangle.
    Let four points (x0, y0), (x1, y1), (x2, y2), (x3, y3)
    As long as the three interior angles are right angles, it can be deduced that these four points form a rectangle
    That is, it is judged that adjacent sides are orthogonal, that is, the edge vector dot product of adjacent sides is 0
    (x3-x0)*(x1-x0)+(y3-y0)*(y1-y0) == 0 (angle with (x0, y0) as vertex)
    and
    (x0-x1)*(x2-x1)+(y0-y1)*(y2-y1) == 0 (angle with (x1, y1) as vertex)
    and
    (x1-x2)*(x3-x2)+(y1-y2)*(y3-y2) == 0 (angle with (x2, y2) as vertex)
    
    value of dot product
    The size of u, the size of v, and the cosine of the angle between u and v. Under the premise that u and v are non-zero, if the dot product is negative, the angle formed by u and v is greater than 90 degrees; if it is zero, then u and v are vertical; if it is positive, then the angle formed by u and v is an acute angle .

    
    @lzylyd provides a way to find out whether two sides are perpendicular through the slope (https://baike.baidu.com/item/%E6%96%9C%E7%8E%87)
    k1 * k2 = (y1-y2)/(x1-x2) * (y2-y3)/(x2-x3) = -1
    


#### 2. Write a piece of code to judge whether there is a ring in the singly linked list. If a ring is formed, please find the entrance of the ring, namely point P
    class Node{
        public $data=null;
        public $next=null;
    }
    
    function eatList(Node $node) {
        $fast = $slow = $node;
        $first_target = null;
        if($node->data == null) {
            return false;
        }
    
        while (true) {
            if($fast->next != null && $fast->next->next != null) {
                $fast = $fast->next->next; //The fast pointer takes two steps at a time
                $slow = $slow->next; //The slow pointer goes one step at a time
            } else {
                return false;
            }
    
            if($fast == $slow) { //The slow pointer catches up with the fast pointer, indicating that there is a loop
                $p1 = $node; //The p1 pointer points to the head node, the p2 pointer points to the point where they intersect for the first time, and then the two pointers move one step at a time, when they intersect again, it is the entry of the ring
                $p2 = $fast;
                while($p1 != $p2) {
                    $p1 = $p1->next;
                    $p2 = $p2->next;
                }
                return $p1; //Entry node of the ring
            }
        }
    }

#### 3. Write a function to get all the pictures in the content of an article and download them
    function downImagesFromTargetUrl($url, $target_dir = null) {
        if(!filter_var($url, FILTER_VALIDATE_URL)){
            return false;
        }
        if(!$target_dir) {
            $target_dir = './download';
        }
    
        $root_url = pathinfo($url);
    
        $html = file_get_contents($url); //main
        preg_match_all('/<img[^>]*src="([^"]*)"[^>]*>/i',$html, $matchs); //main
    
        $images = $matchs[1];
    
        foreach ($images as $img) {
            $img_url = parse_url($img);
            if(!array_key_exists('host', $img_url)) {
                $img_url = $root_url['dirname'] . DIRECTORY_SEPARATOR . $img;
            } else {
                $img_url = $img;
            }
    
            $img_path = array_key_exists('path', $img_url) ? $img_url['path'] : $img;
            $save = $target_dir . DIRECTORY_SEPARATOR . $img_path;
            $save_path = pathinfo($save);
    
            if(!is_dir($save_path['dirname'])) {
                mkdir($save_path['dirname'], 0777, true);
            }
    
            file_put_contents($save,file_get_contents($img_url)); //main
        }
    }

#### 4. Obtain the IP address of the current client and determine whether it is at (1.1.1.1, 255.255.255.254)
    function getip()
    {
        $unknown = 'unknown';
        if (isset($_SERVER['HTTP_X_FORWARDED_FOR']) && $_SERVER['HTTP_X_FORWARDED_FOR'] && strcasecmp($_SERVER['HTTP_X_FORWARDED_FOR'], $unknown)) {
            $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
        } elseif (isset($_SERVER['REMOTE_ADDR']) && $_SERVER['REMOTE_ADDR'] && strcasecmp($_SERVER['REMOTE_ADDR'], $unknown)) {
            $ip = $_SERVER['REMOTE_ADDR'];
        }
        /*
        Dealing with multi-layered proxies
        Or use a regular method: $ip = preg_match("/[\d\.]{7,15}/", $ip, $matches) ? $matches[0] : $unknown;
        */
        if (false !== strpos($ip, ','))
            $ip = reset(explode(',', $ip));
        return $ip;
    }
    
    $client_ip = getip();
    $client_ip = sprintf('%u', ip2long($client_ip)); //64-bit system no pressure
    
    /**
     * plan A
     */
    $range_min = sprintf('%u', ip2long('1.1.1.1'));
    $range_max = sprintf('%u', ip2long('255.255.255.255'));
    
    /**
     * plan B
     */
    $range_min = bindec(decbin(ip2long('1.1.1.1')));
    $range_max = bindec(decbin(ip2long('255.255.255.255')));
    
    
    if ($client_ip >= $range_min and $client_ip <= $range_max) {
        echo 'true';
    } else {
        echo 'false';
    }
#### 5. The log_format configuration of nginx is as follows:
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                           '$upstream_response_time'
                           '$status $body_bytes_sent "$http_referer" '
                           '"$http_user_agent" "$http_x_forwarded_for"';
                           
From today's nginx log file access.log:
* a. List the 20 largest lines of "request_time"?
* b. List the 20 url addresses with more traffic at 10:00 in the morning?


    a: cat /usr/local/var/log/nginx/access.log|sort -nrk9|head -2
    
    b: grep "07/May/2018:10:" /usr/local/var/log/nginx/access.log|awk '{print $12}'|sort -rn|uniq -c|head -20
    

#### 6. What is CSRF attack? XSS attack? How to prevent?
    CSRF: https://baike.baidu.com/item/CSRF/2735433
    Prevention method: CSRF TOKEN, that is, when submitting a form, submit a token generated when the form is rendered by the server, and prevent csrf attacks by verifying the token
    
    XSS: https://baike.baidu.com/item/xss/917356
    To put it simply, XSS is a normal page that executes the front-end code submitted by the user or hacker. For example, if you use eval ('the code submitted by the user is executed here'),
    Or your page parses the html code submitted by the user normally, for example, the personal information submitted by the user is: <imgsrc="Ad link"><script>window.href="Malicious website link"</script>,
    And if you put it into the library without filtering and escaping, and then the page parses the html code normally, the end user will jump to the malicious website when visiting this page, which is XSS
    Prevention method: filter && escape user input (such as htmlentities, htmlspecialchars), never trust the client
    
    

#### 7. In the application, we often encounter the situation of randomly fetching 10 pieces of data in the user table to display, briefly describe how you can implement this function.

    select * from user where rand() limit 10;

#### 8. Randomly draw 5 cards from the poker cards to determine whether it is a straight, that is, these 5 cards are consecutive, JQK is represented by 11, 12, 13
    
    My understanding: Since it is a straight, then there must be no pair. After finding the smallest value, add 1 in order to see if it exists. If both exist, it is a straight.
    If I write it like this:
    function eatChicken(arr $data)
    {
        $min = min($data);
        for ($i = $min; $i < $min + 5; ++$i) {
            if (!in_array($i, $data)) {
                return false;
            }
        }
        return true;
    }
    
    var_dump(eatChicken([1, 3, 5, 2, 4]));
    var_dump(eatChicken([10, 13, 11, 12, 14]));
    var_dump(eatChicken([1, 3, 5, 7, 9]));
    
    https://github.com/hookover/php-engineer-interview-questions/issues/8
    @johson
    function eatDuck(array $arr)
    {
        $count = count($arr);
        if (count(array_unique($arr)) != $count) {
            return false;//pair
        }
        if (max($arr) - min($arr) != $count - 1) {
            return false;
        }
        return true;
    }
    var_dump(eatDuck([1, 3, 5, 2, 4]));
    var_dump(eatDuck([10, 13, 11, 12, 14]));
    var_dump(eatDuck([1, 3, 5, 7, 9]));

    There is a better way, please add, the great gods come to a version without built-in functions
    
#### 9. Two intersecting singly linked lists, how to find their first common node
    https://www.cnblogs.com/cwkpql/p/4743602.html
    https://blog.csdn.net/zuochao_2013/article/details/78447126
    https://blog.csdn.net/jiqiren007/article/details/6572685

#### 10. Longest common subsequence problem LCS, if there are two arrays [1,2,5,11,32,15,77] and [99,32,15,5,1,77], find The numbers they all have in common, write the code with the best time complexity, and cannot use array_intersect (there are pits here, you need to study dynamic programming).
    A * "1+1+1+1+1+1+1+1 =?" *
    
    A : "What is the value of the above equation"
    B : *calculate* "8!"
    
    A *write "1+" on the left side of the equation above *
    A : "What is the value of the equation at this time"
    B : *quickly* "9!"
    A : "How did you know the answer so quickly"
    B : "Just add 1 to 8"
    A : "So you don't have to recalculate because you remember the value of the first equation is 8! The dynamic programming algorithm can also be said to 'remember the solution to save time'""

    https://blog.csdn.net/u013309870/article/details/75193592 (I think the principle is the best)
    https://www.cnblogs.com/zlm-jessie/p/5664562.html

#### 11. The principle of memory allocation and multithreading in linux
    This is still Baidu to find information...

#### 12. The difference between primary key and unique index in MYSQL
    The primary key is a constraint, the unique index is a kind of index, the two are different in nature
    After the primary key is created, it must contain a unique index, and the unique index is not necessarily the primary key
    Unique index columns allow null values, while primary key columns do not allow null values
    When the primary key column is created, it has defaulted to a null value + a unique index
    Primary keys can be referenced as foreign keys by other tables, while unique indexes cannot
    A table can only create at most one primary key, but can create multiple unique indexes
    Primary keys are more suitable for unique identifiers that are not easy to change, such as auto-incrementing columns, ID numbers, etc.
    In RBO mode, the execution plan priority of the primary key is higher than that of the unique index. Both can improve the speed of the query
    
    https://blog.csdn.net/baoqiangwang/article/details/4832814

#### 13. The main difference between http and https
    An understanding based on the OSI model:
    http works at the application layer
    https is http built on the SSL channel, and SSL belongs to the transport layer in the OSI model, so I think HTTPS is a protocol that belongs to the transport layer
    However, some students put forward different opinions. For related discussions, see: https://github.com/hookover/php-engineer-interview-questions/issues/7
    
    So if it is based on the TCP/IP model: HTTP and SSL both work at the application layer, then HTTP and HTTPS are both application layer protocols
    
    http is plaintext transmission
    https is encrypted transmission
    
    The interviewer will ask the difference between ssl digital certificate, symmetric encryption and asymmetric encryption
    
    The Hypertext Transfer Protocol HTTP protocol is used to transfer information between web browsers and web servers. The HTTP protocol sends content in clear text and does not provide any data encryption. If an attacker intercepts the transmission message between the web browser and the web server, he can directly read the information in it. Therefore, the HTTP protocol is not suitable for transmitting some data. Sensitive information, such as credit card numbers, passwords, etc.
    In order to solve this defect of the HTTP protocol, another protocol needs to be used: the Secure Sockets Layer Hypertext Transfer Protocol HTTPS. For the security of data transmission, HTTPS adds the SSL protocol to HTTP. SSL relies on certificates to verify the identity of the server and encrypt the communication between the browser and the server.
    The main differences between HTTPS and HTTP are the following four points:
    1. The https protocol needs to go to ca to apply for a certificate. Generally, there are very few free certificates and you need to pay a fee.
    Second, http is a hypertext transfer protocol, information is transmitted in plaintext, and https is a secure ssl encrypted transfer protocol.
    Third, http and https use completely different connection methods, and the ports used are also different. The former is 80 and the latter is 443.
    Fourth, the connection of http is very simple and stateless; the HTTPS protocol is a network protocol constructed by the SSL+HTTP protocol that can perform encrypted transmission and identity authentication, which is safer than the http protocol.
    
    https://baike.baidu.com/item/https

#### 14. http status code and its meaning
    Basically remember 200, 201, 301, 302, 400, 403, 404, 500, 502, 503

#### 15. How to check system resource usage in linux, and how to check TCP port and TCP connection status
    top //May ask deeper, such as what information is displayed, what information do you care about, viewing a process, etc.
    iostat //disk cpu
    free // memory remaining
    df //disk usage
    du //file occupancy information
    
    ps //View process information
    netstat -anptol //Check the port occupancy, it is recommended to check the document for parameter details, be careful to be asked

#### 16. What is the principle of SQL injection? How to prevent SQL injection
    It is usually the primary code written by junior programmers, which is caused by unfiltered user input. The ORM of modern frameworks is generally processed accordingly. If you need to handle it yourself, there are two solutions:
    1: Escape user input (htmlentities/htmlspecialchars), use the mysql_real_escape_string method to filter the parameters of the SQL statement
    2: precompile sql (best way)

#### 17. $a, $b=null, $c=false, $d="", $e=0, $f=[], what is the output of the above variables using is_null, isset, empty methods respectively?
    $null = null;
    $zero = 0;
    $empty_arr = [];
    $false = false;
    $empty_str = "";
    $not_defintd;
    
    isset($not_defintd) false
    isset($zero) true
    isset($null) false
    isset($false) true
    isset($empty_arr) true
    isset($empty_str) true
    
    empty($not_defintd) true
    empty($zero) true
    empty($null) true
    empty($false) true
    empty($empty_arr) true
    empty($empty_str) true
    
    
    Notice: Undefined variable: not_defintd in /opt/webroot/test.php on line 27
    is_null($not_defintd) true
    is_null($zero) false
    is_null($null) true
    is_null($false) false
    is_null($empty_arr) false
    is_null($empty_str) false

#### 18. How to optimize MYSQL
    Personal understanding:
    We need to talk about optimization from the entire project environment, which can be divided into three aspects:
    Hardware level:
        Adopt high-end sass hard disk, upper disk array
    Architecture level:
        Sub-library, partition, sub-table, master-slave (master-master), multi-server cluster, vip+keepalive, etc. (You may ask about the specific implementation, so you should understand these implementation details before answering)
    Application level (as long as you mention it below, the interviewer may ask for details, such as what storage engines are there, what are the differences and application scenarios, what is the difference between the primary key index and non-primary key index of innodb, data structure, and what is stored in the leaf nodes ?)
    
    Choice of storage engine
        field selection
            shorter is faster
            Fixed-length types are faster than variable-length types
            Integers are processed faster than strings
        index
           Index types supported by MYSQL (when it comes to this, you will definitely ask you for a specific definition)
           Conditions of use of the index
           The implementation structure of the index
               Clustered index (clustered index), non-clustered index, B+Tree
               HASH index
        slow query log
            Can help find the problem statement
        Optimize sql statement by explain
          

#### 19. What is the transaction in the database?
    Characteristics of Transactions: ACID
    Atomicity A set of DML statements either all succeed or all fail
    Consistency transactions must go from one state to another
    Isolation Isolation between multiple transactions can behave differently according to the isolation level of the transaction
    Persistence Durability A committed transaction, once committed, it makes permanent changes to data in the database
    
    Q: What happens when there is no transaction?
    A: When operating the mysql database in the console, if there is no transaction control, misoperation will cause permanent loss of data.
    

Transaction isolation level:
    
|Isolation Level|Dirty Read|Unrepeatable Read|Phantom Read|Locked Read?
|---------|-----|---------|---|---
|Read uncommitted|o|o|o|No lock
|Read committed|x|o|o|No lock
|Repeatable read(Repeatable read)|x|x|o(mysql does not appear x)|No lock
|Serializable|x|x|x|Lock (full table lock)


    Dirty read: When a client queries the modified data that has not been committed by another transaction, it is a dirty read.
    Non-repeatable read: [The same query] is performed multiple times in the [same transaction], and the modification or deletion made by other submitted transactions returns a different result set each time, and a non-repeatable read occurs at this time.
    Phantom read: [The same query] is performed multiple times in the [same transaction]. Due to the insert operations performed by other committed transactions (transactions may not be committed), different result sets are returned each time, and a phantom read occurs at this time.
    
#### 20. Write a function to extract the file extension from a standard URL as efficiently as possible
    The interviewer said to be efficient, I remember it seems that it is not efficient to use regular, so exclude regular? Then he said it is a "standard url", do you want you to use the parse_url function?
    Then I write this:
    $ext = array_pop(explode('.',parse_url($url)['path']));
    I don't think it's efficient
    
#### 21. The parameter is an array of multiple dates and times, and returns the time closest to the current time
    I can only think of the foreach method, welcome to modify
    $data = [
        '2015-02-02 11:11:11',
        '2012-02-02 11:55:11',
        '2019-12-02 11:33:11',
        '2017-12-02 11:22:11',
    ];
    
    $near = array_reduce($data, function($a, $b){
       return abs((time() - strtotime($a))) < abs((time() - strtotime($b))) ? $a : $b;
    });
    
    echo $near;
    
#### 22, the difference between echo, print, print_r
    echo is not a function, there is no return value, it is only used for printing information, it will be faster if only output echo
    print has a return value, is a function, and can format the output
    print_r prints composite types such as array objects
    
#### 23. What are the keys and meanings in the header of the http protocol
    General
        Request URL: http://localhost/test/t.php
        Request Method: GET
        Status Code: 200 OK
        Remote Address: 127.0.0.1:80
        Referrer Policy: no-referrer-when-downgrade
    Reason Headers
        Connection: keep-alive
        Content-Encoding: gzip
        Content-Type: text/html; charset=UTF-8
        Date: Mon, 07 May 2018 10:05:43 GMT
        Server: nginx/1.10.1
        Transfer-Encoding: chunked
        X-Powered-By: PHP/7.0.8
    Request Headers
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
        Accept-Encoding: gzip, deflate, br
        Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
        Cache-Control: no-cache
        Connection: keep-alive
        Host: localhost
        Pragma: no-cache
        Referer: http://localhost/test/
        Upgrade-Insecure-Requests: 1
        User-Agent: Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.117 Mobile Safari/537.36
    
#### 24. Binary tree traversal code
    See the picture of the tree here: https://zhidao.baidu.com/question/235504989.html
    
    class Node
    {
        public $data = null;
        public $left = null;
        public $right = null;
    }
    
    $A = new Node();
    $B = clone $A;
    $C = clone $A;
    $D = clone $A;
    $E = clone $A;
    $F = clone $A;
    $G = clone $A;
    $H = clone $A;
    $I = clone $A;
    
    
    $A->data = 'A';
    $B->data = 'B';
    $C->data = 'C';
    $D->data = 'D';
    $E->data = 'E';
    $F->data = 'F';
    $G->data = 'G';
    $H->data = 'H';
    $I->data = 'I';
    
    
    $A->left = $B;
    $A->right = $C;
    $B->left = $D;
    $B->right = $E;
    $E->left = $G;
    $E->right = $H;
    $G->right = $I;
    $C->right = $F;
    
    /**
     * Preorder traversal: middle left
     * Inorder traversal: left middle right
     * Post-order traversal: left and right middle
     */
    function eatBtree($node)
    {
        if ($node && $node->data) {
            eatBtree($node->left);
            eatBtree($node->right);
            echo $node->data; //Change the position of this line to change the traversal mode, put it at the end is the post-order, put it at the top is the pre-order, put it in the middle is the middle-order
        }
    }
    
    eatBtree($A);
    
    /**
     * Layer order traversal will use the queue
     */
    
    function eatBtree2($node)
    {
        $list[] = $node;
        while (count($list) > 0) {
            $cur = array_shift($list);
            if ($cur) {
                echo $cur->data;
    
                if ($cur->left) {
                    $list[] = $cur->left;
                }
    
                if ($cur->right) {
                    $list[] = $cur->right;
                }
            }
        }
    }
    
    eatBtree2($A);

#### 25. What is the difference between the array structure of PHP and the array structure of C language?
    Definition of C language array: The C language standard stipulates that an array type describes a continuously allocated non-empty collection of objects with a specific element object type. The types of these element objects are called element types. The array type is determined by the element type and the number of elements.
    The data structure of PHP array is implemented by HashTable, and its hash conflict resolution method uses the zipper method. The structure of PHP7's HashTable itself is different from that of 5.
    This topic is not something I can explain very clearly in one or two sentences. It is recommended to check the information, understand the zavl union, and realize the specific way of array, so that when the interviewer asks the question, you can tell it through your own understanding.

#### 26. How is the ordered set implemented in Redis, and what is its data structure?
    https://www.cnblogs.com/paulversion/p/8194966.html

#### 27. What is a hash table? How is the data stored after the hash conflict in the hash table?
    Hash table (also called hash table) is a data structure that is directly accessed according to the key value. That is, it accesses records by mapping the key value to a location in the table to speed up lookups. This mapping function is called a hash function, and the array of records is called a hash table.
    Solution after hash conflict:
        open addressing
        Rehashing
        Chain address method (zipper method)
        Create a common overflow area
    

#### 28. The difference between a clustered index (clustered index) and a non-clustered index

do not?
    https://www.cnblogs.com/Alight/p/3967141.html
    Clustered Index:
        Table data is stored in the order of indexes, that is, the order of index entries is consistent with the physical order of records in the table. For a clustered index, the leaf nodes store the actual data rows, and there are no separate data pages. Only one clustered index can be created on a table at most, because there can only be one physical order of real data.
        Appears only on the primary key index of the innodb engine.
                Clustering: the primary key index key of innodb, which is stored together with the record
                
                resulting in:
                Records are sorted by primary key order
                The real location of the record will change. Changes as the primary key index key changes
                Then the non-primary key index (secondary index) on the innodb table: what is stored is: the correspondence between keywords and primary key values ​​(instead of the correspondence between keywords and record positions)
                Result: non-primary key index of innodb: all secondary search.
                1 keyword determines the primary key value.
                2 The primary key value determines the record
                
                At the data structure level:
                In the original B-Tree structure, some changes have been made, and the changed cluster (cluster) structure is called B+Tree
                
    Non-clustered index:
        Table data storage order is independent of index order. For a non-clustered index, the leaf node contains the index field value and a logical pointer to the data row of the data page, and the number of rows is the same as the data volume of the data table row.

    

    Update table data:
        Insert a new row of data into the table
           If a table does not have a clustered index, it is called a "heap". The rows of data in such a table are in no particular order, and all new rows will be added to the end of the table. The data table with a clustered index is different: in the simplest case, the insert operation finds the corresponding data page according to the index, then moves the existing records to make room for the new data, and finally inserts the data. If the data pages are full, the data pages need to be split, the index pointers adjusted (and if the table has nonclustered indexes, these indexes need to be updated to point to the new data pages). Similar to auto-incrementing as a clustered index, the database system may not split data pages, but simply add new data pages.
        delete data row from table
           For deletion of data rows: Deleting a row will cause the data row below it to move up to fill the gap created by the deleted record. If the deleted row is the last row in the data page, then the data page will be reclaimed and the records in the corresponding index page will be deleted. For data deletion, there may be only one record in the index page. At this time, the record may be moved to the adjacent index page, and the original index page will be recycled, which is called "index merge".
        
    A clustered index is a sparse index, and the index page one level above the data page stores the page pointer, not the row pointer. For a non-clustered index, it is a dense index, which stores an index record for each data row on the upper-level index page of the data page.

#### 29. How does B+Tree search
    Others have specialized research: https://blog.csdn.net/hguisu/article/details/7786014
#### 30. What is the difference between an array and a hash table?
    An array is a data type provided by programming languages, that is, a group of contiguous memory spaces are used to store data, and any location in this group of memory spaces can be directly accessed through a first address and an array subscript. search
    Hash table is a concept in the discipline of data structure. It uses an array as a storage method to realize a data structure that can quickly find data. It is to pass the value of the data through a mapping function to obtain a result, and then put the data at the position of the array subscript corresponding to the result. search
        
#### 31. Write a function to determine whether the following extension is closed or not. Left and right symmetry is closed: ((())), )(()), (()))), (((((()) ), (()()), ()()
    When a left bracket is encountered, it is pushed into the stack, and when a right bracket is encountered, it is popped out of the stack (if there is none in the stack, it means that it is not closed), traverse to the last element, and judge that the stack is empty, that is, it is closed.
    
    function checkClose($str)
    {
        $stack = [];
    
        for ($i = 0; $i < strlen($str); ++$i) {
            if ($str[$i] == "(") {
                $stack[] = "(";
            }
    
            if ($str[$i] == ")") {
                $border = array_pop($stack);
    
                if(!$border) {
                    return false;
                }
            }
        }
    
        if (count($stack) == 0) {
            return true;
        }
        return false;
    }
    
    var_dump(checkClose('(())'));
    var_dump(checkClose('(())()(('));
    var_dump(checkClose('(())()()'));
    var_dump(checkClose('(())()))'));
    var_dump(checkClose('(5+2)*6/(3-1)'));

#### 32. Find the unique values ​​in the array [1,2,3,3,2,1,5]
    The idea of ​​using hash/bucket
    $res = [];
    foreach ($data as $item) {
        if(array_key_exists($item, $res)) {
            ++$res[$item];
        } else {
            $res[$item] = 1;
        }
    }
    
    foreach ($res as $k=>$v) {
        if($v == 1) {
            echo $k;
        }
    }
    

#### What is your time complexity for questions 33 and 32? In some cases, you write an algorithm, and the interviewer asks you to write the time complexity expression of your algorithm
     O(n+m) -> O(n) ?

#### 34. How is this weak type variable of PHP implemented?
    zval complex

#### 35. During the HTTP communication process, is the client or the server actively disconnecting?
    Look here: https://www.cnblogs.com/web21/p/6397525.html
    
#### 36. What are the ways to initiate HTTP requests in PHP? How are they different?
    curl
    way of stream
    socket method
    https://segmentfault.com/a/1190000010302052
    
#### 37. There is a binary tree, write code to find out the shortest path from the root node to the flag node and print it out, there are multiple flag nodes. For example, 6 and 14 in the tree below are flag nodes, please write the code to print 8, 3, 6 and 8, 10, 14 paths
![](https://gss2.bdstatic.com/9fo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=b3c80026d72a6059761de948495d5ffe/94cad1c8a786c9179df930bedcf6c93.jpg)jpg

#### 38. There are two files, the size is more than 1G, one line of data, each line of data does not exceed 500 bytes, part of the content of the two files is exactly the same, please write the code to find the same line, and write to a new file. The maximum allowable size of PHP is 255M.
    It seems to be another large file processing. The interviewer's intention to ask the question does not want you to traverse the two-layer for loop. This answer is definitely not necessary. Look at the connection of the 64 questions, and specifically talk about various ways of processing big data. .
    Then borrowing the plan of the article, my solution to this problem is as follows:
        Read all the records of the two files in sequence, and store each record in 10 files after hash->converting to decimal->%n, so that a total of 2G of data is divided into 10 copies, each of which is 204.8M , below the memory limit,
        I can read one file at a time and use the hash bucket method to get whether the content in a single file is duplicated, because each record is hashed, so the same record will definitely be in the same file.
        
        Here is pseudo code:
        
        /**
         * Store each record in the two files in 10 files after hashing the remainder
         * If a file is too large and exceeds the limit memory size, it can be hashed again
         */
        $handler = fopen('file_a_AND_file_b', 'r');
        
        while ($line = fgetc($handler)) {
            $save_to_file_name = crc32(hash('md5', $line)) % 10;
            file_put_contents($save_to_file_name, $line);
        }
        
        /**
         *
         */
        $files = [ 'Paths to 10 files' ];
        foreach ($files as $file) {
        
            $handler = fopen($file, 'r');
            $tmp_arr = [];
            while($line = fgetc($handler)) {
                if(isset($tmp_arr[$line])) {
                    file_put_contents('common_content.txt', $line);
                } else {
                    $tmp_arr[$line] = true;
                }
            }
        
        }
    
    
#### 39. Please write at least two PHP functions that support callback processing, and implement a PHP function that supports callbacks by yourself
    
    array_reduce();
    array_map();
    array_filter();
    
    function callBack($parameter, $fn) {
        return $fn($parameter);
    }
    
    var_dump(callBack(5, function ($n){
        return $n * $n;
    }));
    
#### 40. Please write at least two methods (codes or ideas) to get all the files in the specified folder.
    //recursive
    function readDirDeep($path, $deep = 0)
    {
        $handle = opendir($path);
        while(false !== ($filename = readdir($handle))){
            if($filename == '.' || $filename == '..') continue;
            echo str_repeat('&nbsp;',$deep*5) . $filename.'<br>';
                //str_repeat(str,n) repeats a str string n times
            if(is_dir($path.'/'.$filename)){
                readDirDeep($path.'/'.$filename,$deep+1);
                }
            }
            //close
            closedir($handle);
    }
    
    //queue
    The way to queue is to put it into the queue when it encounters a directory, and print it if it is not a directory.
    function readDirQueue($dir)
    {
        $dirs = [$dir];
    
        while ($path = array_shift($dirs)) {
            if (is_dir($path) && $handle = opendir($path)) {
                while (false !== ($filename = readdir($handle))) {
                    if ($filename == '.' || $filename == '..') continue;
                    $real_path = $path . DIRECTORY_SEPARATOR . $filename;
    
                    if(is_dir($real_path)) {
                        $dirs[] = $real_path;
                    }else {
                        echo $real_path . '<br/>';
                    }
                }
                //close
                closedir($handle);
            }
        }
    
    }

#### 41. Please write at least three methods or functions for intercepting file name suffixes (PHP native functions and self-implemented functions can be used)
    $file = 'x.y.z.png';
    $ext = substr(strrchr($file, '.'), 1);
    $ext = pathinfo($file)['extension'];
    $ext = array_pop(explode('.', $file));
    
    https://blog.csdn.net/zls986992484/article/details/52629684
    
#### 42. How does PHP deliver cookies to clients without using its own cookie function. For distributed systems, how to save session values.
    1. You can use the page to directly output the cookie, and the client js writes it, such as:
    <?php
        $cookie = 'abcd...';
        "<script> setcookie($cookie); </script>"
    ?>
    2. Through JSON data transmission, JS front-end saves, such as:
    <?php
        json_encode(['cookie'=>'abcd...']);
    ?>
    <html><body><script>
        ajax{
            success: function(data){
                var cookie = data.cookie;
            }
        }
    </script></body></html>
    
    2. Distributed system session storage: mysql, redis, memcache, files, the main method is that all app applications operate the session in the same location, which can be stored anywhere, depending on the volume of business. For example, if the volume of business is large, it may Using a cache cluster, the business volume is small, and the files of a single machine may be stored.
    The specific implementation scheme can be searched on google/baidu, there are many cases
    
    
#### 43. Please use SHELL to count the most visited URL addresses in the nginx log within 5 minutes. What are the corresponding IPs?
    The workload is a bit heavy, please wait
#### 44. Write a shell script to back up the mysql specified library (such as test) to the specified folder and package it, delete the backup 30 days ago, and then push the new backup to the remote server, and send an email notification after completion .
    The workload is a bit heavy, please wait
#### 45. The difference between innodb and myisam engine in mysql database
    InnoDB:
    Support transaction processing, etc.
    Unlocked read
    Support foreign keys
    support row lock
    Indexes of type FULLTEXT are not supported
    Do not save the exact number of rows in the table, scan the table to calculate how many rows there are
    When DELETE the table, it is a row-by-row deletion
    InnoDB stores data and indexes in tablespaces
    Cross-platform can be directly copied and used
    InnoDB must contain indexes for fields of type AUTO_INCREMENT
    Tables are hard to compress
    
    MyISAM:
    Does not support transactions, rollback will cause incomplete rollback, not atomic
    Foreign keys are not supported
    Foreign keys are not supported
    Support full text search
    Save the specific number of rows in the table, without where, directly return the number of saved rows
    When DELETE table, drop the table first, then rebuild the table
    MyISAM tables are stored in three files. The frm file holds table definitions. The data file is MYD (MYData). The index file is an extension of MYI (MYIndex)
    It is difficult to copy directly across platforms
    In MyISAM, you can make a joint index for AUTO_INCREMENT type fields
    Tables can be compressed
    
    Highlights:
    After MYSQL5.7, all the functions that INNODB did not have have been perfected, that is, INNODB has all the advantages of MYISAM, and myisam will be abandoned in the upcoming MYSQL8.0 version
    
#### 46. From the user entering the URL in the browser and pressing Enter, to seeing the complete page, what processes have been experienced in the middle.
     Browser->url->dns->ip->port->tcp->nginx->server name->php-fpm/fast cgi->php
       ^ <- client ip:port <- ^ <- ^ <-
       
     The whole process will probably involve these, and the details can be understood
     
     By the way: what is fast cgi? What is the relationship between php and php-fpm?
    
#### 47. How to analyze the performance of a SQL statement.
    Familiar with the various properties of the response after the explain analysis statement

#### 48. If ping a server fails to ping, which command is used to trace routing packets?
    linux:traceroute,windows:tracert
    
#### 49. $a=[0,1,2,3]; $b=[1,2,3,4,5]; $a+=$b; how much is var_dump($a) equal to?
    The test is the difference between array + and array_merge
    When the subscript is a numeric value, array_merge() will not overwrite the original value, but the array+array merge array will return the first value as the final result, and "abandon" those values ​​with the same key name in the following arrays (not overwrite).
    When the subscript is a character, array+array still returns the first value as the final result, and "abandons" those values ​​with the same key name in the following array, but array_merge() will overwrite the previous value with the same key name at this time. value.
    

#### 50, $a=[1,2,3]; foreach($a as &$v){} foreach($a as $v){} var_dump($a) equals; (I added )
    https://laravel-china.org/articles/7001/php-ray-foreach-and-references-thunder

#### 51. The user ID is stored in the database, and there are many lines of deductions. Redis stores the user's wallet. Now we need to write a script to synchronize the deduction records in the database to redis and execute it every 5 minutes. once. What issues should be considered?
    First of all, I haven't actually done this project. If I do it, it will be like this:
    
    I will save the user's balance as an integer, for example, 1 yuan is represented by 100 points
    The deduction record in MYSQL will have a time to update the data
    
    First get each pending user:
       select uid from deduction record table where update_time='0000-00-00 00:00:00' group by uid;
       
    Process each user sequentially:
        open business
            Handling deduction records in MYSQL
                $time = date('Y-m-d H:i:s'); //current time
                update deduction record table set status='processing status code',update_time=$time where uid=single user ID;
                {
                    This can be optimized to update all users' 5-minute data at a time, and then process them one by one:
                    update deduction record table set status='processing status code',update_time=$time where update_time='0000-00-00 00:00:00'
                }
            
                select sum (deduction amount column) from deduction record table where uid=single user ID and update_time=$time; //time is the key
            
            Processing the REDIS wallet:
                DECRBY decrements the given value
                decrby key number
        End the transaction (ensure redis, the deduction is successful, then submit)
    
    
    
#### 52. MYSQL master-slave server, if the master server is the innodb engine and the slave server is the myisam engine, what problems will be encountered in practical applications?
    Totally inexperienced...
    
#### 53. What are the process signals in linux?
    $kill -l
    1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL
     5) SIGTRAP 6) SIGABRT 7) SIGBUS 8) SIGFPE
     9) SIGKILL 10) SIGUSR1 11) SIGSEGV 12) SIGUSR2
     13) SIGPIPE 14) SIGALRM 15) SIGTERM 17) SIGCHLD
    18) SIGCONT 19) SIGSTOP 20) SIGTSTP 21) SIGTTIN
    22) SIGTTOU 23) SIGURG 24) SIGXCPU 25) SIGXFSZ
    26) SIGVTALRM 27) SIGPROF 28) SIGWINCH 29) SIGIO
    30) SIGPWR 31) SIGSYS 34) SIGRTMIN 35) SIGRTMIN+1
    36) SIGRTMIN+2 37) SIGRTMIN+3 38) SIGRTMIN+4 39) SIGRTMIN+5
    40) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8 43) SIGRTMIN+9
    44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
    48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13
    52) SIGRTMAX-12 53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9
    56) SIGRTMAX-8 57) SIGRTMAX-7 58) SIGRTMAX-6 59) SIGRTMAX-5
    60) SIGRTMAX-4 61) SIGRTMAX-3 62) SIGRTMAX-2 63) SIGRTMAX-1
    64) SIGRTMAX
    
    I only remember one: kill -9 process id...
#### 54. The underlying implementation of redis

#### 55. Asynchronous model
    Personal understanding, there may be deviations, please correct me:
    User A tells CPU to read file a, and CPU immediately tells user A: OK, wait for me, then hand over the request to read file a to the disk, and register a callback event (you have to tell me after reading it)
    User B tells the CPU to read the b file, and the CPU immediately tells the B user: OK, you wait for me, and then hand over the request to read the b file to the disk, and register a callback event (you have to tell me after reading it)
    b The amount of data in the file is small. After reading, the callback function is triggered to report the returned data to the CPU, and the CPU hands the data to user B.
    After a is read, the callback function is triggered to report the returned data to the CPU, and the CPU hands the data to user A
    
    Although AB is waiting, but my big CPU has not stopped, and AB users have received the first response, please correct me.
    
   
    It's very detailed here, I haven't read it seriously: https://tech.youzan.com/yi-bu-wang-luo-mo-xing/
    
#### 56, 10g file, use php to view its line number
    From the network: Its way is to read a part of the data at a time, calculate how many newlines in this part of the data, and keep looping, the efficiency will be higher than reading the content sequentially
    /*
     * Efficiently calculate the number of file lines
     * @author axiang
    */
    function count_line($file)
    {
        $fp = fopen($file, "r");
        $i = 0;
        while (!feof($fp)) {
            //Read 2M each time
            if ($data = fread($fp, 1024 * 1024 * 2)) {
                // Calculate the number of lines read
                $num = substr_count($data, "\n");
                $i += $num;
            }
        }
        fclose($fp);
        return $i;
    }
#### 57. If there are 1 billion order data belonging to 1000 drivers, please take out the drivers with the top 20 orders
    The idea the other party wants:
    1. Read 1 billion pieces of data sequentially, and count the number of orders for each driver at each point
    2. Build a max heap, filter 1000 drivers sequentially, and find the top 20 drivers
    The pseudo-code:
    $order_data = [];
    foreach(1 billion orders as $order_info) {
        if(isset($order_data[$order_info]) {
            $order_data[$order_info] ++;
        } else {
            $order_data[$order_info] = 1;
        }
    }
    
    $map = [];
    foreach ($order_data as $num) {
        if (count($map) < 20) {
            $map[] = $num;
            continue;
        }
        $min = min($map);
        if ($num > $min) {
            for ($i = 0; $i < count($map); ++$i) {
                if ($map[$i] == $min) {
                    $map[$i] = $num; //replace the minimum value
                    break; //Break out of the loop, only replace once
                }
            }
        }
    }
    
    
#### 58. Design a function of WeChat red envelope (detailed implementation details from code, server architecture, database, performance, etc.)
    WeChat red envelope system design sharing, how to withstand 10 billion requests: http://www.woshipm.com/pd/232838.html
    Knowing that there are some algorithms: https://www.zhihu.com/question/22625187
    QA: https://www.zybuluo.com/yulin718/note/93148
    
    Note: For example, if you are asked to write two interfaces, one for grabbing red envelopes and one for sending red envelopes, you can design a complete set of storage system and code implementation. How do you do it?
    
#### 59. According to the access.log file, count the qps of the last 5 seconds, and display it in the following format, the number of n seconds is such as: 01 1000 (the difficulty lies in the number of 01 seconds)
    error.log data:
    2018/07/29 03:16:01...  
    2018/07/29 03:16:01...  
    
    awk '{print $1,$2}' access.log|sort -nr|awk '{t[$1" "$2]++} END {for(i in t){print i,t[i]}}'| sort -nrk 1,2|head -20|cut -c18-

#### 60. Why is the performance of php7 improved so high?
    https://laravel-china.org/articles/6201/questions-and-answers-that-laravel-and-phper-interviews-may-encounter
    The difference between PHP7 and PHP5, what new features are added?
        Twice the performance
        Combining Comparison Operators (<=>)
        Scalar Type Declaration
        return type declaration
        try...catch adds multiple conditional judgments, and more Error errors can be handled with exceptions
        Anonymous classes, which now support instantiating an anonymous class via new class, which can be used to replace some "burn-in" full class definitions
        …Learn more at the bottom of the article with links to new features in PHP7
    Why is PHP7 faster than PHP5?
        Variable storage bytes are reduced, memory usage is reduced, and variable operation speed is improved
        Improve the array structure, the array elements and the hash mapping table are allocated in the same block of memory, which reduces the memory usage and improves the cpu cache hit rate
        The function calling mechanism has been improved, and some instructions have been reduced by optimizing the parameter transmission link to improve the execution efficiency.
        
    
    http://coffeephp.com/articles/4?utm_source=laravel-china.org
    10. php7 new features#
        ?? operator (NULL coalescing operator)
        function return type declaration
        Scalar Type Declaration
        use bulk declaration
        define can define an array of constants
        Closure adds a call method. Details can be found on the official website: php7-new-features
    11. The optimization behind the excellent performance of php7#
        Reduce the number of memory allocations
        Use more stack memory
        Cache the hash value of the array
        String parsing into eucalyptus instead of macro expansion
        Use large pieces of contiguous memory instead of small pieces of broken memory For details, please refer to Brother Bird's PPT: The Source of PHP7 Performance

#### 61. Traverse a multidimensional array
    $data = [
        1, 2, 4, 6,
        [
            2, 5, 6, 7,
            [
                9, 12, 55, 66, 77
            ]
        ],
    ];
    
    function eatArr(array $arr)
    {
        foreach ($arr as $item) {
            if (is_array($item)) {
                eatArr($item);
            } else {
                echo $item . ' ';
            }
        }
    }
    
    eatArr($data);
    
#### 62. There is such a string abcdefgkbcdefab... random length, write a function to find the number of times bcde appears in this string
    $str = "abceeedefdsafujdsklgjmrj89gu89eeefiodsaflkjdsafjhuigbeeejhndfiofgidsafndyeeeubhngihaf;odsa";
    $tag = "eee";
    
    function search($str, $need)
    {
        $res = [];
        $str_len = strlen($str);
        $need_len = strlen($need);
        for ($i = 0; $i < $str_len; ++$i) {
            for ($n = 0; $n < $need_len; ++$n) {
                if (isset($str[$i + $n]) && $need[$n] != $str[$i + $n]) {
                    b
                    break;
                }
                if ($n == $need_len - 1) {
                    $res[] = $i;
                }
            }
        }
        return $res;
    }
    
    var_dump(search($str, $tag), count(search($str, $tag)));

    Three methods are compared here, and the traversal is the fastest: https://blog.csdn.net/yxtyxt3311/article/details/599492
    
#### 63. There is a file with a size of 1G, each line in it is a word, the size of the word does not exceed 16 bytes, and the memory limit size is 1M. Return the 100 most frequent words
    Well said: https://blog.csdn.net/zzran/article/details/8443655
    
#### 64. Ten interview questions for massive data processing and a summary of ten methods (added by me)
    https://blog.csdn.net/v_JULY_v/article/details/6279498

#### 65. The php process model, how does php support multiple concurrency

#### 66. The process model of nginx, how to support multiple concurrency

#### 67. The meaning of each configuration of php-fpm, the daemonize mode of fpm
    static - the number of child processes is fixed (pm.max_children)
    ondemand - processes are spawned when there is demand (when requested, as opposed to dynamic, pm.start_servers is started when the service starts
    dynamic - the number of child processes is dynamically set based on the following configurations: pm.max_children, pm.start_servers, pm.min_spare_servers, pm.max_spare_servers

#### 68. Let you implement a simple architecture and maintain high availability. There are two interfaces, one to upload a text and one to get the uploaded content. How do you design it? It is necessary to avoid single-machine room failures, and at the same time, make the code level insensitive.
    
#### 69. Two mysql servers, one of which is down, how to make the business side switch without feeling, and ensure that the data of the podium server is consistent under normal circumstances
    
#### 70. The specific definition of the http protocol

#### 71. What is lock and how to solve the problem of lock
    How is the lock formed?
    For example, there are two files: a.php and b.php
    The operation of a.php is: first lock the A record, and then lock the B record. Only when the B record is locked will the lock of the A record be released.
    The operation of b.php is: first lock the B record, and then lock the A record. Only when the A record is locked will the lock of the B record be released.
    
    At this time, a.php locks the A record, and b.php locks the B record
    When a.php finishes processing the A record, it wants to lock the B record, but it turns out that the B record is locked, so the lock added to the A record cannot be released
    The b record is also unable to release the lock of the B record because the A record cannot be locked
    This creates a deadlock
    
    How to deal with it:
    Both a.php and b.php should be changed to the same locking sequence
    a.php first locks A and then locks B, then b.php also locks A and then locks B
    
#### 72. The difference between rand and mt_rand
    1.int rand(void) / int mt_rand(void)
    2. int rand(int $min, int $max) / int mt_rand($min, $max)
     
    For the first form:
         The random number generated by rand() is between 0 and getrandmax()
         The random number generated by mt_rand() is between 0 and mt_getrandmax()
     
    For the second form:
         rand() generates random numbers from $min to $max
         mt_rand() generates random numbers from $min to $max
     
    Compared:
         mt_rand() is a better random number generator because it sowns a better random number seed than rand(); and is 4 times faster in performance than rand() for the range of values ​​represented by mt_getrandmax() also bigger
         
#### 73. How is mysql transaction isolation implemented?
    https://www.linuxidc.com/Linux/2018-01/150610.htm
    https://liuzhengyang.github.io/2017/04/18/innodb-mvcc/
    
#### 74. How to realize the lock of mysql
    I read an article before, but I can't find the link, I will post it later
    
    It is related to the storage structure, and the index will be locked (just like the file lock, there is a lock mark), if there is no lock index, the table will be locked
    In addition, the lock will have a range, the gap lock
    
    https://www.cnblogs.com/luyucheng/p/6297752.html
#### 75. Symmetric encryption and asymmetric encryption
    Better on the search engine, check it out yourself
    
    I understand:
    Than both parties get the answer through the same key and algorithm decryption, that is, symmetric encryption a<<2=xxx xxx >>2 = a Symmetric encryption
    If A encrypts aaa with a to get ccc, and B decrypts ccc with b to get aaa, this is asymmetric encryption
    
#### 76. 10 bottles of water, one of which is poisonous. After the mice drink the poisonous water, they will die after 24 hours. Q: It takes at least a few mice to find out which bottle it is after 24 hours. Water is poisonous.
    @saurfang Representing 10 in binary requires at most a few digits. 2^A>=10,A=4
![](images/mouse.jpg)

#### 77. How does redis synchronize, the method of synchronization, what to do with synchronization rollback, and what to do with data exceptions. At the same time, it will ask about the synchronization method of MYSQL and related exceptions
    
#### 78. How to solve cross-domain

#### 79, the difference between json and xml, what are the advantages and disadvantages of each

#### 80, Trait priority

#### 81. A refers to b, and the page reports an error: the class in c is repeatedly defined, and the content of the abc3 pages is written. What will happen to the circular reference?

#### 82. The salary of employee 3 below is greater than that of his supervisor, and a SQL finds the supervisor whose salary is lower than that of his subordinates
    
|id |username|salary|pid|
|------|------|---------|-----:|
|1 | a | 3000 | null |
|2|b|8000|null|
|3|c|5000|1|
|4|d|6000|3|
    
    select b.id,b.username
    from employee as a
    left join employee as b
    on a.manger_id=b.id
    where a.salary>b.salary
    group by b.id;

#### 82. There is a polygon consisting of N points in a coordinate system, and now there is a coordinate point, write code or ideas to determine whether this point is within the polygon
    http://www.html-js.com/article/1517
    The blogger wrote 3 solutions
    One of them is probably that when the ray traverses the boundary is odd, it is inside the polygon, otherwise it is outside the polygon, but to judge the special case (such as at the vertex)

#### 83. If there is a deadlock in the database, how do you check and how do you judge that there is a deadlock?
    SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;
    SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
    
    SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
    
    SHOW processlist;
    SHOW ENGINE INNODB STATUS;
    

#### 84.1. Write a program to find the longest substring

#### 84.2. Write a program to find the longest common consecutive substring

#### 84.3, find the longest repeating substring in a string

#### 85. Analyze a problem: The log of php-fpm is normal, but the client has timed out. What do you think may be the problem? How to troubleshoot?

#### 86. What is the workflow of nginx, which can be described by drawing

#### 87. What are the methods of inter-process communication?
    https://blog.csdn.net/violet_echo_0908/article/details/51201278
    1 Anonymous pipe communication
    2 Advanced Pipeline Communication
    3 Named Pipe Communication
    4 Message Queue Communication
    5 Semaphore communication
    6 Signals
    7 Shared memory communication
    8 Socket communication

#### 88. Master-slave replication, will the slave server read the data that the master server is rolling back? The master database is successfully written, but the slave server fails to write for some reasons. What will happen in the end? Master-slave replication what if there is a key conflict?

#### 89. How many isolation levels are there for transactions? How is the isolation level of a transaction implemented?
    
[19. What is the transaction in the database] (#19, What is the transaction in the database?)

    http://tech.it168.com/a2016/0905/2900/000002900122.shtml
    https://liuzhengyang.github.io/2017/04/18/innodb-mvcc/
    https://blog.csdn.net/matt8/article/details/53096405
    
#### 90. What is B+ number, please draw the structure of b+ tree (expand other trees)
    
    https://blog.csdn.net/v_JULY_v/article/details/6530142
    http://www.cnblogs.com/yangecnu/p/Introduce-B-Tree-and-B-Plus-Tree.html

#### 91.1, the character set in mysql, the client is inconsistent with the database, what should I do?
    
    Modify the default character set
    (1) The simplest modification method is to modify the character set key value in the my.ini file of mysql,
         Such as default-character-set = utf8
          character_set_server=utf8
         After the modification, restart the mysql service
    (2) Another way to modify the character set is to use the mysql command
         mysql> SET character_set_client = utf8;
         mysql> SET character_set_connection = utf8;
         mysql> SET character_set_database = utf8;
         mysql> SET character_set_results = utf8;
         mysql> SET character_set_server = utf8;
         mysql> SET collation_connection = utf8;
         mysql> SET collation_database = utf8;
         mysql> SET collation_server = utf8;

#### 91.2, MYSQL Chinese characters

What is the process of character conversion from string to display to interface?

    A complete user-requested character set conversion process is
       1) When mysql Server receives the request, it converts the request data from character_set_client to character_set_connection
       2) Convert the request data from character_set_connection to the internal operation character set before performing the internal operation, the steps are as follows
            A. Use the CHARACTER SET settings for each data field;
            B. If the above value does not exist, use the character set of the corresponding data table to set the value
            C. If the above value does not exist, use the character set setting value of the corresponding database;
            D. If the above value does not exist, use character_set_server to set the value.
       3) Finally convert the operation result from the internal operation character set to character_set_results
     
![](images/charset.jpg)
 

#### 91.3. The character set in the database is latin1. Now you store the utf8 character string in the database table of the latin1 character set. Can you store the utf8 character string in it?
    By default (characters such as client, connection, etc. are all utf8):
        English numbers and symbols can be used, but not Chinese.
        insert into latin1(data) values('123abc!@#'); //Success, display normal
        insert into latin1(data) values('Chinese 123abc!@#'); //failure, 1366 error
        
        because:
            latin1 is single byte
            utf8 is three bytes
            Chinese requires 2 bytes to store
        
    Special cases (the client character set is the same as the table character set, such as utf8):
        At this time, both Chinese and English can be inserted successfully
        insert into latin1(data) values('123abc!@#'); //Success, display normal
        insert into latin1(data) values('Chinese 123abc!@#'); //Success, display garbled characters

    
#### 91.4. If you say it can be saved, ask: Can it be restored? If so, how to restore it?
    When inserting data, the database will first convert the incoming string to the corresponding character set type in the table, and then store it in the database.
    
    So when the following statement is executed:
        insert into latin1(data) values('Chinese 123abc!@#'); //Success, display garbled characters
   
    The client converts "Chinese 123abc!@#" into a Latin string, such as (ä¸æ–‡123abc!@#), and sends it to the service MYSQL server,
    The MYSQL server will judge whether the character set of ä¸æ–‡123abc!@# can be stored in the database, find OK, and store it in

    recover:
    After testing: I have not found a recovery method for the time being
    

#### 92. Write a piece of code to find all subsets, such as [a,b,c] subsets are {},{a},{b},{c},{ab},{ac}, {abc}
    http://plutoblog.iteye.com/blog/976218 (principle)
    https://my.oschina.net/liuhui1990/blog/40422
    https://blog.csdn.net/a568283992/article/details/53525253 (sum of squares of subsets)
    
    // recursive method
    function allSubSet(array $arr = [], $sub_set = "", $begin_point)
    {
        $res = [];
        if($sub_set) {
            $res[] = "{" . trim($sub_set, ",") . "}";
        }
        for ($start_point = $begin_point; $start_point < count($arr); ++$start_point) {
            $res = array_merge(
                $res,
                allSubSet($arr, $sub_set . "," . $arr[$start_point], $start_point + 1)
            );
        }
    
        return $res;
    }
    
    $data = allSubSet([1, 2, 3], '', 0);
    var_dump($data);

#### 93. ['a'=>200,'b'=>100,'c'=>100], write a custom sorting function, descending by value, if the values ​​are the same, sort by key
    function kvsort($arr) {
        $res_arr = [];
        while (count($arr)) {
            $min = null;
            $min_key = null;
    
            foreach ($arr as $key=>$value) {
                if(!$min || $min > $value) {
                    $min = $value;
                    $min_key = $key;
                } else if($min == $value && $min_key > $key) {
                    $min = $value;
                    $min_key = $key;
                }
            }
            unset($arr[$min_key]);
            $res_arr[$min_key] = $min;
        }
        return $res_arr;
    }

#### 94. Design a cache system that can automatically delete long-term unused data periodically or after the space is full, and cannot be used for traversal.
    My answer at the time was to use a linked list for storage. If the cache hits, the cache is moved to the head of the linked list, and then the tail of the linked list is full of cold data.
    I remember where I saw this design before, but I forgot to connect it, please know if my friend posted the link.
    
    https://www.cnblogs.com/zengzy/p/5167827.html
    
####95, the difference between == and ===, write the following output: "aa"==1, "bb"==0, 1=="1"

#### 96. A sorted array, cut it into two arrays from any position in the middle, then exchange their positions and merge them. After the merger, the new array elements are: 20, 21, 22, 25, 30, 1, 2, 3, 5, 6, 7, 8, 15, 18, 19, write a query function to find whether a certain value exists.
    Think first, then fill in the answer when you have time
    An optimized version of 2-point search
    First find the middle number and divide the data into 2 segments,
    Determine whether the leftmost value is less than the rightmost value (the two pieces of data are judged separately, first right and then left, and no judgment is required for hits)
        If it is true, it means that this part of the data is sequential. If the data is in this interval, it will continue to search according to the 2-point algorithm.
        If it is false, it means that this part of the data is not sequential, then continue to cut into 2 segments, repeat until there is only the last element or can not find it
    
    

#### 97. Design a tree structure, and then write a function to traverse it in level order
    24 questions
    https://github.com/hookover/php-engineer-interview-questions#24%E4%BA%8C%E5%8F%89%E6%A0%91%E5%89%8D%E4%B8%AD% E5%90%8E%E9%81%8D%E5%8E%86%E4%BB%A3%E7%A0%81

#### 98, the difference between '$var' and "$var"
    Escape characters and variables in double quotes in PHP can be parsed, but single quotes cannot
    
#### 99, the difference between self and static

    static: If the static methods and properties in the parent class are rewritten in the subclass, the parent class will access the static methods of the subclass
    self: is a pointer within the class, regardless of whether the subclass has rewritten the methods and attributes in the parent class, it points to the static methods and attributes of this class
    
    http://php.net/manual/en/language.oop5.late-static-bindings.php

#### 100, PHP coroutines and uses
    About coroutines, the most you may see is the phrase "coroutines are user-mode threads".
    To understand what a "user-mode thread" is, you must first understand what a "kernel-mode thread" is. Kernel-mode threads are scheduled by the operating system. When switching thread contexts, the context of the previous thread must be saved first, and then the next thread will be executed. When the conditions are met, switch back to the previous thread and restore the context. The same is true for coroutines, except that user-mode threads are not scheduled by the operating system, but by the programmer, which is in user-mode.
    The keyword yield is used to generate an interrupt and save the current context. For example, a piece of code in the program is to access a remote server. At this time, the CPU is idle, use yield to yield the CPU, and then execute the next piece of code. If the next piece of code still accesses other resources than the CPU, you can also call yield to give up the CPU. Continue to execute, so that you can write asynchronous code in a synchronous way.
    https://segmentfault.com/q/1010000009144366
    https://yq.aliyun.com/articles/53673
    

#### 101. Describe the mechanism of autoload
    https://blog.csdn.net/zhihua_w/article/details/52723402
    
#### 102. How many bytes each field type in mysql: smallint, int, bigint, datetime, varchar(8)
    • TINYINT - a tiny integer, supports -128 to 127 (SIGNED), 0 to 255 (UNSIGNED), requires 1 byte of storage
    • BIT - same as TINYINT(1)
    • BOOL - same as TINYINT(1)
    • SMALLINT - a small integer, supports -32768 to 32767 (SIGNED), 0 to 65535 (UNSIGNED), requires 2 bytes to store MEDIUMINT - a medium integer, supports -8388608 to 8388607 (SIGNED), 0 to 16777215 ( UNSIGNED), requires 3 bytes of storage
    • INT - an integer, supports -2147493648 to 2147493647 (SIGNED), 0 to 4294967295 (UNSIGNED), requires 4 bytes of storage
    • INTEGER - same as INT
    • BIGINT - a large integer, supports -9223372036854775808 to 9223372036854775807 (SIGNED), 0 to 18446744073709551615 (UNSIGNED), requires 8 bytes of storage
    • FLOAT(precision) – a floating point number. precision<=24 for single-precision floating point numbers; precision is between 25 and 53 for high precision floating point numbers. FLOAT(X) has the same range as the corresponding FLOAT and DOUBLE types, but does not define the display size and the number of decimal places. Before MySQL 3.23, this was not a true floating point value and always had two decimal places. All calculations in MySQL use double precision, so this can cause some unexpected problems.
    • FLOAT - a small menu-precision floating point number. Supports -3.402823466E+38 to -1.175494351E-38, 0 and 1.175494351E-38 to 3.402823466E+38, requires 4 bytes of storage. If UNSIGNED, the range of positive numbers remains unchanged, but negative numbers are not allowed.
    • DOUBLE - a double-precision floating-point number. Supports -1.7976931348623157E+308 to -2.2250738585072014E-308, 0 and 2.2250738585072014E-308 to 1.7976931348623157E+308. In the case of FLOAT, UNSIGNED does not change the range of positive numbers, but negative numbers are not allowed.
    • DOUBLE PRECISION - same as DOUBLE
    • REAL - same as DOUBLE
    • DECIMAL - store a number as a string, one byte per character
    • DEC - same as DECIMAL
    • NUMERIC - same as DECIMAL
    
    String column types: char, varchar, nvarchar
    String column types are used to store any type of character data, such as names, addresses, or newspaper articles. Below are the string column types available in MySQL
    • CHAR—Character. A fixed-length string, padded with spaces on the right, to the specified length. Supports from 0 to 155 characters. When searching for values, trailing spaces are removed.
    • VARCHAR - variable length character. A variable-length string with trailing spaces removed when storing the value. Supports characters from 0 to 255
    • TINYBLOB - Tiny binary object. 255 characters are supported. Requires length + 1 byte of storage. Same as TINYTEXT, except that searches are case-sensitive. (0.25KB)
    • TINYTEXT - supports 255 characters. Length + 1 byte of storage is required. Same as TINYBLOB, except case is ignored when searching. (0.25KB)
    • BLOB - binary object. 65535 characters are supported. Requires length + 2 bytes of storage. (64KB)
    • TEXT - 65535 characters are supported. Length + 2 bytes of storage are required. (64KB)
    • MEDIUMBLOB - medium size binary object. 16777215 characters are supported. Requires length + 3 bytes of storage. (16M)
    • MEDIUMTEXT - supports 16777215 characters. Requires length + 3 bytes of storage. (16M)
    • LONGBLOB - Large binary object. 4294967295 characters are supported. Requires length + 4 bytes of storage. (4G)
    • LONGTEXT - supports 4294967295 characters. Requires length + 4 bytes of storage. (4G)
    • ENUM - Enumeration. There can only be one specified value, i.e. NULL or "", with a maximum of 65535 values
    • SET - a set. There can be 0 to 64 values, all from the specified list.
    
    
    
    Date and time column types
    Date and time column types are used to handle temporal data, and can store data such as the time of day or the date of birth. Format specification: Y represents the year, M (before M) represents the month, D represents the day, H represents the hour, M (the last M) represents the minute, and S represents the second. Below are the date and time column types available in MySQL
    • DATETIME - Format: 'YYYY-MM-DD HH:MM:SS', Range: '1000-01-01 00:00:00' to '9999-12-31 23:59:59'
    • DATE - Format: 'YYYY-MM-DD', Range: '1000-01-01' to '9999-12-31'
    • TIMESTAMP - Format: 'YYYYMMDDHHMMSS', 'YYMMDDHHMMSS', 'YYYYMMDD', 'YYMMDD', Range: '1970-01-01 00:00:00' to '2037-01-01 00:00:00'
    • TIME - Format: 'HH:MM:SS'
    • YEAR - Format: 'YYYY, Range: '1901' to '2155'


#### 103. Which attributes uniquely determine a TCP connection

#### 104. The difference between myisam and innodb, why is myisam faster than innodb, what is the index data structure of myisam and innodb? The difference between innodb primary key index and non-primary key index? What is the data stored on the index? ?
    45 questions
    
    Clustered index (clustered index), B+Tree:
    
        Appears only on the primary key index of the innodb engine.
        Clustering: the primary key index key of innodb, which is stored together with the record
        
        resulting in:
        Records are sorted by primary key order
        The real location of the record will change. Changes as the primary key index key changes
        Then the non-primary key index (secondary index) on the innodb table: what is stored is: the correspondence between keywords and primary key values ​​(instead of the correspondence between keywords and record locations)
        Result: non-primary key index of innodb: all secondary search.
        1 keyword determines the primary key value.
        2 The primary key value determines the record
        
        At the data structure level:
        In the original B-Tree structure, some changes have been made, and the changed cluster (cluster) structure is called B+Tree

    Fast query speed:
        When INNODB is doing SELECT, there are many more things to maintain than the MYISAM engine:
        1) Data blocks, INNODB needs to be cached, MYISAM only caches index blocks, and there is a reduction in swapping in and out.
        2) The innodb addressing needs to be mapped to the block and then to the line. MYISAM records the OFFSET of the file directly, and the positioning is faster than INNODB.
        3) INNODB also needs to maintain MVCC consistency; although your scene does not have it, it still needs to be checked and maintained
        MVCC (Multi-Version Concurrency Control) multi-version concurrency control
        
        Notes:
        InnoDB: Implements MVCC by adding two additional hidden values ​​to each row, one to record when the row of data was created, and the other to record when the row of data expires (or is deleted). But InnoDB does not store the actual time when these events occurred, instead it only stores the system version number when these events occurred. This is a number that keeps growing as transactions are created. Each transaction records its own system version number at the beginning of the transaction. Every query must check whether the version number of each row of data is the same as the version number of the transaction. Let's see how this strategy applies to specific operations when the isolation level is REPEATABLEREAD:
        SELECT InnoDB must per-row data to ensure that it meets two conditions:
        1. InnoDB must find a row version that is at least as old as the transaction's version (that is, its version number is not greater than the transaction's version number). This ensures that the row of data exists before the transaction starts, or when the transaction is created, or when the row of data is modified.
        2. The deleted version of this row of data must be undefined or larger than the transaction version. This ensures that the row of data is not deleted before the transaction starts
        

#### 105. When the TCP connection is disconnected, the timewait state will appear on the end that initiates the breakup or the end that is broken up

#### 106. There are a lot of data analysis tests in AWK. You need to practice more, and the questions will not be written one by one.
    https://segmentfault.com/a/1190000009745139
    http://wiki.jikexueyuan.com/project/awk/

#### 107. What is the data structure of collection, ordered collection, hyperLog, and hash in redis
    Remark: You need to understand the implementation of ordered sets, including skip table structure and implementation

#### 108. Describe: the entire processing process of a request reaching nginx (what logic will be called by nginx itself), then how to communicate with php, what is the process in the middle, etc.?
    https://blog.csdn.net/yankai0219/article/details/8220695
    shocking herd
    By the way, learn about epoll, select, poll
   
#### 109, the related configuration of nginx and php-fpm, just ask what the various parameters mean
    https://www.baidu.com/s?wd=nginx%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%AF%A6%E8%A7% A3
    https://www.baidu.com/s?wd=php-fpm%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E8%AF%A6%E8% A7%A3
    
#### 110. If there is a map, as shown in the figure below, "-" represents the ocean, and "+" represents the land. Use your best method to get the coordinates of the largest land area.
    --++----++--+++---
    -++++----+++++++--
    -+++----++++------
    -----++------++++-
    ---++++++-----+++-
    -----+++------+++-
    
    Backtracking, depth-first traversal
    https://leetcode-cn.com/problems/max-area-of-island/

#### Basic Algorithms: Bubble, Fast, Cask, Binary Search
    Find it online

####php implements a function that converts decimal to binary
    Converting decimal integers to binary integers adopts the method of "divide by 2 and take the remainder and arrange in reverse order".
    The specific method is: divide the decimal integer by 2, you can get a quotient and a remainder; then divide the quotient by 2, you will get a quotient and a remainder, and so on, until the quotient is 0,
    Then take the remainder obtained first as the low significant bit of the binary number, and the remainder obtained later as the high significant bit of the binary number, and arrange them in sequence.
    789=1100010101
    789/2=394 remainder 1 10th
    394/2=197 more than 0 9th place
    197/2=98 remainder 1 8th
    98/2=49 remainder 0 7th place
    49/2=24 remainder 1 6th place
    24/2=12 remainder 0 5th place
    12/2=6 remainder 0 4th digit
    6/2=3 remainder 0 3rd digit
    3/2=1 remainder 1 second digit
    1/2 get 0 more than 1 1st place
    
    function dec2bin($num)
    {
        if (!is_int($num)) return false;
        $bin = '';
        while ($num > 1) {
            $bin .= $num % 2;
            $num = ($num - $num % 2) / 2;
        }
        return strrev($bin . $num);
    }
    
    echo dec2bin(3);
    
#### The number of processes of php-fpm is more reasonable
    https://segmentfault.com/a/1190000000630270
    https://gist.github.com/hookover/3c8c958026cc09dcbaa258c06db9a3ef
    https://www.kinamo.be/en/support/faq/determining-the-correct-number-of-child -processes-for-php-fpm-on-nginx

#### The encryption process of https?
    https://developer.huawei.com/ict/forum/thread-47495.html
    One-way authentication (client without certificate):
        1. Client: Send client SSL version information
        2. Server: The server returns the SSL version, random number and other information to the client, as well as the server public key
        3. Client: The client verifies whether the server certificate is legal, and continues legally, otherwise an alarm
        4. Client: The client improves the symmetric encryption scheme that it can support to the server for selection
        5. Server: The server selects the encryption method with the highest encryption program
        6. Server: Send the selected encryption method to the client in clear text
        7. Client: After receiving the encryption method, generate a random code as a symmetric encryption key, encrypt it with the server's public key, and send it to the server
        8. Server: The server uses the private key to decrypt the encrypted information to obtain the symmetric encryption key
        n. Data channel: form a symmetric encrypted channel to ensure communication security
        
    Two-way authentication (the client has a certificate):
        1. Client: Send client SSL version information
        2. Server: The server returns the SSL version, random number and other information to the client, as well as the server public key
        3. Client: The client verifies whether the server certificate is legal, and continues legally, otherwise an alarm
        4. Client: After the client passes the verification, it sends its own certificate and public key to the server
        5. Server: verify the client certificate, and obtain the client's public key after the verification
        6. Client: The client improves the symmetric encryption scheme it can support to the server for selection
        7. Server: The server chooses the encryption method with the highest encryption program
        8. Server: send the selected encryption scheme [encrypt with client public key] to the client
        9. Client: After receiving the encryption method, use the private key to decrypt, generate a random code, as a symmetric encryption key, encrypt it with the server's public key, and send it to the server
        10. Server: The server decrypts the encrypted information with the private key and obtains the key for symmetric encryption
        n. Data channel: form a symmetric encrypted channel to ensure communication security
    
    Client has certificate

#### Other people's interview answers
    http://coffeephp.com/articles/4?utm_source=laravel-china.org
    https://laravel-china.org/articles/6844/a-php-interview-for-a-16-year-old-graduate
    https://todayqq.gitbooks.io/phper/content/

#### resource collection
    "A Collection of Various Algorithms Implemented in PHP" https://github.com/PuShaoWei/arithmetic-php
    "The Art of Programming: Interviews and Algorithms" https://wizardforcel.gitbooks.io/the-art-of-programming-by-july/content/index.html
    "Sword Finger OFFER-PHP Implementation" https://blog.csdn.net/column/details/15795.html
    "PHP version of Leecode, five questions per week" https://github.com/wuqinqiang/leetcode-php
