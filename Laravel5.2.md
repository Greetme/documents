# `Laravel 5.2`

## 1 `Laravel` 基础篇

### 1.1 `Laravel` 安装


### 1.2 `Laravel` 路由
```
// 基本路由
// GET请求
Route::get('basic1', function () {
    return 'Hello World';
});
// POST请求
Route::post('basic2', function () {
    return 'Basic2';
});

// 多请求路由
// 响应指定的多个 HTTP 请求
Route::match(['get', 'post'], 'multy1', function () {
    return 'multy1';
});
// 响应所有 HTTP 请求
Route::any('multy2', function () {
    return 'multy2';
});

// 路由参数
// 必选参数
Route::get('user/{id}', function ($id) {
    return 'User-id-' . $id;
});
// 可选参数
/*Route::get('user1/{name?}', function ($name = null) {
    return 'User-name-' . $name;
});*/
Route::get('user1/{name?}', function ($name = 'John') {
    return 'User-name-' . $name;
});
// 用表达式验证参数
Route::get('user2/{name?}', function ($name = 'John') {
    return 'User-name-' . $name;
})->where('name', '[A-Za-z]+'); // 验证 name 字段匹配0到多个大小写字母
// 多参数验证
Route::get('user3/{id}/{name?}', function ($id, $name = 'John') {
    return 'User-id-' . $id . '-name-' . $name;
})->where(['id' => '[0-9]+','name' => '[A-Za-z]+']);

// 路由别名
Route::get('user4/member-center', ['as' => 'center', function () {
    //return 'member-center';
    return route('center'); // 返回 'http://demo.laravel52.com/index.php/user4/member-center'
}]);

// 路由群组
Route::group(['prefix' => 'member'], function () {
    // 访问 member/user41/member-center
    Route::get('user41/member-center', ['as' => 'center', function () {
        return route('center');
    }]);

    // 访问 member/multy2
    Route::any('multy2', function () {
        return 'member-multy2';
    });
});

// 路由中输出视图
Route::get('view', function () {
    return view('welcome');
});
```

### 1.3 `Laravel` MVC
#### 1.3.1 模型 Model
如 /app/User.php
#### 1.3.2 视图 View
如 /resources/views/member/info.blade.php
#### 1.3.3 控制器 Controller
如 /app/Http/Controllers/Controller.php

### 1.4 数据库操作之 `DB facade`
#### 1.4.1 连接数据库
编辑两个文件：/config/database.php 和 /.env  
(1) /config/database.php 编辑：
```
'default' => env('DB_CONNECTION', 'mysql'),
```
(2) /.env 编辑对应参数：
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

#### 1.4.2 使用 `DB facade` 实现 CURD
```
// 读取数据  
$student = DB::select('SELECT * FROM student WHERE id > ?', [1001]);

// 新增数据，返回bool<br/>
$bool = DB::insert('INSERT INTO student(name, age) VALUES(?, ?)', ['imooc', 19]);

// 更新数据，返回修改的行数  
$num = DB::update('UPDATE student SET age = ? WHERE name = ?', [20, 'sean']);

// 删除数据，返回删除的行数  
$num = DB::delete('DELETE FROM student WHERE id > ?', [1001]);
```

### 1.5 数据库操作之 `查询构造器 query builder`
使用 `PDO` 参数绑定，以保护应用程序免于 `SQL注入`，因此传入的参数不需要额外转义特殊字符。

#### 1.5.1 `查询构造器`新增数据
```
// 新增单条数据，返回bool
$bool = DB::table('student')->insert(
	['name' => 'imooc', 'age' => 18]
);

// 新增数据时，获取主键id
$id = DB::table('student')->insertGetId(
	['name' => 'sean1', 'age' => 17]
);

// 新增多条数据，返回bool
$bool = DB::table('student')->insert([
	['name' => 'name1', 'age' => 17],
	['name' => 'name2', 'age' => 18]
]);
```

#### 1.5.2 `查询构造器`更新数据
(1) 更新为指定内容
```
// 更新为指定内容，返回影响的行数
$num = DB::table('student')->where('id', 1002)->update(['age' => 17]);
```
(2) 自增和自减
```
// 自增和自减，返回影响的行数

// 所有行自增1
$num = DB::table('student')->increment('age');
// 所有行自增2
$num = DB::table('student')->increment('age', 2);
// 所有行自减
$num = DB::table('student')->decrement('age', 5);

// 指定行自增减，并更新其他数据
$num = DB::table('student')->where('id', 1002)->increment('age', 5, ['name' => 'iimooc']);
```

#### 1.5.3 `查询构造器`删除数据
```
// (1) delete

// 删除所有行,返回删除的行数
$num = DB::table('student')->delete();

// 删除指定行,返回删除的行数
$num = DB::table('student')->where('id', 1006)->delete();
$num = DB::table('student')->where('id', '>=', 1024)->delete();
var_dump($num);


// (2) truncate,清空数据表,不返回任何值,慎用
// DB::table('student')->truncate();
```

#### 1.5.4 `查询构造器`查询数据
(1) get()
```
// get() 读取所有数据
$student = DB::table('student')->get();
```
(2) first()
```
// first() 读取第一条数据
$student = DB::table('student')->orderBy('id', 'desc')->first();
```
(3) where()
```
// where() 条件查询
$student = DB::table('student')->where('id', '>=', 1002)->get();

// whereRaw() 多条件查询
$student = DB::table('student')->whereRaw('id >= ? and age > ?', [1003, 14])->get();
```
(4) pluck()
```
// pluck() 读取结果集指定字段值
$names = DB::table('student')->pluck('name');
```
(5) lists()
```
// list() 读取结果集指定字段值,可以指定如 id 作为键名
$names = DB::table('student')->lists('name', 'id');
```
(6) select()
```
// select() 读取指定字段
// $student = DB::table('student')->select('id', 'name', 'age')->get();
```
(7) chunk()
```
// chunk() 读取组块结果集(闭包，处理成千上万条数据库记录)
//echo '<pre>';
// 如分段每次读取2条数据
DB::table('student')->chunk(2, function($student){
	var_dump($student);
	
	// 通过从闭包函数中返回false来中止组块的运行
	/* if (条件) {
		// return false;
	} */
});
```

#### 1.5.5 `查询构造器`聚合函数
cunt()，max()，min()，avg()，sum()
```
// count() 统计条数
$num = DB::table('student')->count();

// max()，min() 最值
$max = DB::table('student')->max('age');
$min = DB::table('student')->min('age');

// avg() 平均数
$avg = DB::table('student')->avg('age');

// sum() 求和
$sum = DB::table('student')->sum('age');
```

### 1.6 数据库操作之 `Eloquent ORM`

#### 1.6.1 `Eloquent ORM` 模型的建立及查询数据
(1) 模型的建立  
创建模型如 /app/Student.php
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Student extends Model
{
    // 指定表名，默认为“模型名 + s”,如 students
    protected $table = 'student';

    // 指定id，默认即为 id
    protected $primaryKey = 'id';
	
	// 指定允许批量赋值的字段
    protected $fillable = ['name', 'age'];

    // 指定不允许批量赋值的字段
    //protected $guarded =[];

    // 自动生成时间戳，默认true
    //public $timestamps = false;

    /**
     * 获取当前时间戳（重写方法）
     * @return int
     */
    protected function getDateFormat()
    {
        return time();
    }

    /**
     * 返回不做处理的时间戳（重写方法）
     * @param mixed $value
     * @return mixed
     */
    protected function asDateTime($value)
    {
        return $value;
    }
}
```


(2) 查询数据  
① all()、find()、findOrFail()，在控制器方法里编写：
```
// all() 查询表的所有记录，返回集合 Collection
$students = Student::all();

// find() 根据主键查询
$students = Student::find(1001);

// findOrFail() 根据主键查询，如果没有记录则报错
$students = Student::findOrFail(1006);
```

② 查询构造器在 ORM 中的使用
```
// 查询构造器在 ORM 中的使用
$students = Student::get();
$students = Student::where('id', '>=', 1002)->orderBy('age', 'desc')->first();

// chunk()
Student::chunk(2, function($students){
    var_dump($students);
});

// 聚合函数
$num = Student::count();
$max = Student::where('id', '>', 1001)->max('age');
```

#### 1.6.2 `Eloquent ORM` 新增数据、自定义时间戳及批量赋值
```
// (1) 使用模型新增数据：如果模型 Student 未重写方法 getDateFormat()，则生成格式化时间，否则为时间戳
$student = new Student();
$student->name = 'imooc2';
$student->age = 18;
$bool = $student->save();


// (2) 自定义时间戳：如果模型 Student 未重写方法 asDateTime()，则为格式化时间，否则为时间戳
$student = Student::find(1035);
$created_at = $student->created_at;
echo date('Y-m-d H:i:s', $created_at); // 自定义格式化时间


// (3) 批量赋值：需要模型 Student 重写属性 $fillable 如 protected $fillable = ['name', 'age']
// 使用模型的 Create() 方法新增数据
$student = Student::create(
	['name' => 'immoc3', 'age' => 19]
);

// firstOrCreate() 通过属性读取数据, 如果不存在则新增并读取新的实例
$student = Student::firstOrCreate(
	['name' => 'immoc123']
);

// firstOrNew() 通过属性读取数据, 如果不存在则创建新的模型实例（并没有持久化到数据库中），需要调用 save() 方法手动持久化到数据库中
$student = Student::firstOrNew(
	['name' => 'immoc1234']
);
$bool = $student->save();
```

#### 1.6.3 `Eloquent ORM` 更新数据

#### 1.6.4 `Eloquent ORM` 删除数据
