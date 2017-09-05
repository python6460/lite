# Grammar Overview
    // Export
    xpt Overview;

    // Import
    mpt System;

    // Variable，当可以明确判断类型时，:type 可以省略。例如 vr a = "10";
    vr a = "10" :string;
    vr c = 1.2 :number;
    vr d = true :bool;

    // Invariable
    invr e = 2;

    // Array
    vr arr = [1,2,3,4,5] :[number];
    print( arr[0] ); //使用下标获取

    // Dictionary, 前面为key，后面为value
    vr dic = ["1":false, "2":true] :[string:bool];
    print( dic["1"] ); //使用key获取

    // Tuple
    vr tpl1 = (1, 2) :(number, number); 
    vr tpl2 = (a:1, b:2);

    // Function
    vr e = (number) -> (number){} :(number)->(number); 

    //无参数无返回值函数
    invr b2c = () -> ()
    {
        A;
        B;
    };

    //无名称完整函数，函数第一个括号为元组参数，箭头后的为元组返回值
    invr c2d = (number, string) -> (number, string)
    {
        rcv (x, y);  //使用rcv来接收参数
        rtn (x, y);  //使用rtn来返回结果
    };

    //带名称完整函数，返回值如参数一样，会自动初始化。
    invr a2b = (x:number, y:string) -> (a:number, b:string)
    {
        a = x;
        b = y;
        //如果不需要做指定的退出，rtn可以省略。
    };

    b2c();
    //使用 _ 舍弃返回值，必须要接收返回值
    _ = a2b(x:3, y:"test");

    // Judge，符合条件的箭头内容被执行，然后自动跳出，无法手动控制
    jg x
    {
        0...6 ->
            A;
        14 ->
            B;
        _ -> //缺省执行，省略的话找不到匹配会自动跳出
            C;
    };

    //当参数的结果只有bool时，相当于if，只当true时才执行
    jg 1+1 != 2
    {
        A;
    };

    //当没有参数时，只执行条件为true的语句，代替if-elseif-else
    jg
    {
        1+1 != 2 -> 
            A;
        3-3 > 0 -> 
            B;
    };

    // Loop，如果不需要获取值，箭头部分可以省略
    lp array -> item
    {
        A;
        B;
    };

    //范围循环
    lp 0...num
    {
        A;
        B;
    };

    //当没有参数时，无限循环
    lp
    {
        A;
        B;
        jg a > b 
        {
            brk; //使用brk跳出循环
            //cnt; //使用cnt跳过当前循环
        };
    };

    // Package Head，属性集合，只能定义一次
    pkg hd Animal //hd只支持vr 非函数类型
    {
        type = "";
        age = 0;
        name = "";
    };

    // Package Fragment，方法集合，可以定义多次，只要方法不存在重名
    pkg frg Animal //frg只支持vr 函数类型
    {
        cry = () -> () 
        {
            A;
            B;
        };
        sleep = () -> ()
        {
            A;
            B;
        };
    };

    pkg hd Pet
    {
        name = "";
    };

    pkg frg Pet
    {
        sleep = () -> ()
        {
            A;
            B;
        };
    };

    //Combine Package，通过引入来复用属性和方法
    pkg hd Dog
    {
        ~Animal; //引入包
        type = ""; //重名自动覆盖，这会代替包原本的属性
        ~Pet;    //当包的属性名称唯一时，编译器自动继承。否则不能通过同名直接使用，要么通过子属性访问，要么手动指定名称。
        name~Pet.name; //手动重新指定继承
        // Implement protocol property
        ~Run
        {
            speed = 0;
        };
    };

    pkg frg Dog
    {
        cry = () -> () //若包有重名，自动覆盖
        {
            slf.Animal.cry(); //手动重载方法，slf用来指定自身
        };
        sleep~Pet.sleep; //手动重新指定继承
        // Implement protocol method
        ~Run
        {
            move = (s: (number) -> (string) )  -> ()
            {
                invr t = 5000/Run.HIGHSPEED; //调用协议公开常量
                invr txt = s.down(time: t);
                print(txt);
                Run.breath(); //调用协议公开方法
            };
        };
    };

    // Protocol, implemented by package
    ptcl Run
    {
        vr speed = 0; //协议支持变量，在包头中实现
        invr HIGHSPEED = 10;   //协议的不变量，必须在协议中初始化，作为公共属性给任意地方使用，只读
        invr breath = () -> (string) //协议的不变量方法，必须在协议中初始化，作为公共方法给任意地方使用，只读
        {
            rtn "ha";
        };
        vr move = (s: (number) -> (string)) -> (){}; //协议的方法，在包片段中实现
    };

    // Create an object
    vr d1 = Dog{};
    // Calling object property
    d1.name = "Tiny";
    // Calling object method
    d1.cry();
    _ = d1.breath();

    // Lambda Function
    d1.move(s: _(time:number) -> (txt:string)
    { 
        invr t = time;
        txt = t :string + "ok";
    });

    // Defer function
    dfr
    {
        A;
    };

    // Declare an Excption Function
    vr readFile = (url:string) -> (data:string)
    {
        jg url.length < 1
        {
            // Report some error
            rpt ErrorInput("URL is none"); 
        };
        data = url;
    };

    // Check, listen the Excption Function
    chk
    {
        readFile(url:"./test.xy");
    }
    -> ErrorInput // Catch the report
    {
        slf.exit(0);
    };

    // Use $ to declare async function
    vr task = $(in:number) -> (out:number)
    {
        // Use .. to make a function to await
        .. A; 
        B;
        .. C; 
        out = in;
    };