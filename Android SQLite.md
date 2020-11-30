1、存储方式简介
      之前已经讲解过APP对接服务器的demo，众所周知，一个APP，必不可少地会涉及到服务器以及数据库的操作。连接服务器，可以让我们合理地利用服务器以及客户端进行任务分配，如果所有功能都在客户端或者服务端完成，必然会造成一头大一头小的情况。合理地安排任务，能够大大地提高APP的稳定性以及可操作性。而对于一个多用户使用的APP，用户使用过程更是会产生许多的数据，其中一些有用的数据需要我们保存下来，因为下次使用用户可能还需要继续使用。

      在安卓开发之中，提供给我们使用的存储方法有较多。

如果这些数据只是在当前activity生命周期之中有用到的（一次性数据），我们可以直接通过定义变量的形式来临时储存（activity结束或者系统内存紧张时，这些变量随时都有可能被回收）。这种方法优缺点比较明显，存储快速，定义简单，但是数据没有持久性。
将数据存储在手机文件里。将数据存储在手机文件里，可以解决上面临时变量的缺点，利用JAVA流将数据存储在手机文件之中，对于一些数据类型较为简单的存储十分实用。但是以文件的形式来存储数据，缺点也很明显，首先，手机文件存储数据，能实现的存储数据类型十分有限，对于一些具有复杂联系的数据难以储存，就算存放进去了，后期的维护也是十分艰难。
第三个存储方法就是利用JAVA提供的一个SharedPreferences类来存储数据。其实这一种方法跟手机文件存储有点相似，但是该方法下存储的数据是以键值对的形式进行存储的，之后在读取时可以通过键来读取对应键的内容，这样的存储方式对于一些常见的键值数据的存储十分方便。虽然SharedPreferences方法比之手机文件存储方式多了一个键值对的功能，但是仍然无法很好地处理一些复杂数据类型。
数据库。数据库相信大部分人都有听说过，数据库分为多种类型，而最常见的就是关系型数据库了。关系型数据库通过数据之间的关系来建表，并且通过表的形式来管理数据，实现数据的增删查改。虽然数据库能够很好地维护一些复杂数据类型，但是对于一些简单的数据存储，就没有必要使用数据库来管理了，这样反而显得冗余。
2、SQLite简介  
    本次程序使用的数据库是SQLite数据库，实现的是一个APP连接到数据库的增查改操作。先看一下APP实现的功能：



     SQLite是一款轻量级的关系型数据库，它运算速度快，占用资源少，通常只需要几百k的内存就够了，支持标准的sql语法和数据库的ACID事务。在Android中为了能够更加方便的管理数据库，专门提供了一个SQLiteOpenHelper帮助类，借助这个类就可以非常简单的对数据库进行创建和升级。



     SQLiteOpenHelper是一个抽象类，如果我们要使用的话，就需要创建一个自己的类去继承它。SQLiteOpenHelper中有2个抽象方法，分别是onCreate和onUpgrade方法，我们必须在自己的帮助类里面重写这两个方法，然后分别在这两个方法中实现创建，升级数据库的逻辑。



     SQLiteOpenHelper中还有2个非常重要的实例方法，getReadableDatabase()和getWritableDatabase()方法。这两个方法都可以创建或打开一个现有的数据库（如果数据库已经存在则直接打开，否则创建一个新的数据库），并返回一个可对数据库进行读写操作的对象。不同的是当数据库不可写入的时候（如磁盘空间已满）getReadableDatabase()方法返回的对象将以只读的方式去打开数据库，而getWritableDatabase()方法方法将出现异常。



     SQLiteOpenHelper中有2个构造方法可供重写，一般使用参数少点的那个构造方法即可。这个构造方法中接收4个参数，第一个参数是Context，必须要有Context才能对数据库进行操作。第二参数是数据库名，创建数据库时使用的就是这里使用的名称。第三个参数允许我们在查询数据的时候返回一个自定义的Cursor，一般都是传如null。第四个参数表示当前数据库的版本号，可用于对数据库进行升级操作。构建出SQLiteOpenHelper的实例后，在调用它的getReadableDatabase()或getWritableDatabase()方法就能够创建数据库了，数据库文件会存放在/data/data/<package name>/databases/目录下。此时，重写的onCreate方法也会得到执行，所以通常在这里去处理一些创建表的逻辑。

3、SQLite数据库的创建
      SQLite数据库的创建主要是通过继承SQLiteOpenHelper并重写其onCreate(用来创建数据库)，onUpgrade(用来更新数据库)，onDowngrade(用来数据库的版本向下兼容，若是不重写该方法，那么就必须保证数据库的版本version每次操作都是大于或者等于上一次的操作的版本)。具体看一下代码：

public class MySQLite extends SQLiteOpenHelper{
 
	//从ShareData中取得想要新建的表名
	ShareData sData=new ShareData();
	//建表语句
	private Context context;
	public  MySQLite(Context context,String name,
			SQLiteDatabase.CursorFactory factory,int version) {
		super(context,name,factory,version);
		this.context=context;
	}
	/*
	 * 建表，该表有四列，分别为主键id、username、password以及验证码verification
	 * @see android.database.sqlite.SQLiteOpenHelper#onCreate(android.database.sqlite.SQLiteDatabase)
	 */
	@Override
	public void onCreate(SQLiteDatabase db) {
		if(!sData.get().isEmpty()) {
		String CREATE_BOOK="create table "+sData.get()+"("+"id integer primary key autoincrement,"
			        +"username text,"+"password text,"+"verification integer)";
		db.execSQL(CREATE_BOOK);
		Log.i("建表界面","表名是"+sData.get());
		Toast.makeText(context, "建表成功", Toast.LENGTH_SHORT).show();
		}
		else
		Toast.makeText(context, "获取表名失败", Toast.LENGTH_SHORT).show();	
	}
	@Override
	public void onUpgrade(SQLiteDatabase db,int oldVersion,int newVersion) {
		if(oldVersion!=newVersion) {
		db.execSQL("drop table if exists "+sData.get());
		Log.i("更新界面","表名是"+sData.get());
		onCreate(db);
		}
	}
	@Override
	public void onDowngrade (SQLiteDatabase db, int oldVersion, int newVersion) {
 
		
	}
	/*
	 * 判断某张表是否存在的方法
	 */
	/**
     * 判断某张表是否存在
     * @param tabName 表名
     * @return
     */
    public boolean tabIsExist(String tabName){
            boolean result = false;
            if(tabName == null){
                    return false;
            }
            SQLiteDatabase db = null;
            Cursor cursor = null;
            try {
                    db = this.getReadableDatabase();//此this是继承SQLiteOpenHelper类得到的
                    String sql = "select count(*) as c from sqlite_master where type ='table' and name ='" + tabName.trim() + "' "; 
                    cursor = db.rawQuery(sql, null);
                    if(cursor.moveToNext()){
                            int count = cursor.getInt(0);
                            if(count>0){
                                    result = true;
                            }
                    }
                     
            } catch (Exception e) {
                    // TODO: handle exception
            }                
            return result;
    }
}
      其实上面代码写了那么多，真正用于创建数据库的就是这两句

String CREATE_BOOK="create table "+sData.get()+"("+"id integer primary key autoincrement,"
			        +"username text,"+"password text,"+"verification integer)";
		db.execSQL(CREATE_BOOK);
      第一句是创建数据库的SQL语言，这个SQL命令通过db.execSQL发送执行，这样当数据库接收到建表指令之后就会执行建立数据库的操作。当然还需要在mainActivity里面实例化该类，然后进行一次getWritableDatabase方法，该方法下，若是数据库存在将会打开数据库，若是数据库不存在则会执行上面的onCreate方法建立新的数据库。
mSqLite=new MySQLite(this, "QiuRuibo_SQLite", null
					,version);
			Log.i("MainVersion",""+version);
			sData.set(creatTableText.getText().toString());
			mSqLite.getWritableDatabase();
       MainActivity的代码：

public class MainActivity extends BaseActivity implements OnClickListener{
 
	private Button creatTableButton,next;
	private EditText creatTableText;
	private MySQLite mSqLite;
	private SharedPreferences sPreferences;
	private SharedPreferences.Editor editor;
	ShareData sData;
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		//初始化控件
		creatTableButton=(Button)findViewById(R.id.newTableButton);
		next=(Button)findViewById(R.id.nextButton);
		creatTableText=(EditText)findViewById(R.id.newTableText);
		creatTableButton.setOnClickListener(this);
		next.setOnClickListener(this);
		//从ShareData中设置想要新建的表名
		sData=new ShareData();
		//实例化一个shared来保存数据
		sPreferences=(SharedPreferences)getSharedPreferences("MySQLiteVersion", MODE_PRIVATE);
		//使用Editor类来存放数据到SharedPreferences中
		editor=sPreferences.edit();
		
		creatTableText.setFilters(new InputFilter[]{filter});
		
	}
	/*
	 * 禁止editText输入空格或者回车，因为数据库建表时不允许空格或者回车等特殊字符
	 */
	private InputFilter filter=new InputFilter() {
		   @Override
		   public CharSequence filter(CharSequence source, 
				   int start, int end, Spanned dest, int dstart, int dend) {
		      if(source.equals(" ")) {
		    	  Toast.makeText(MainActivity.this, "表名不允许出现空格", Toast.LENGTH_SHORT).show();
		    	  return "";
		      }
		      else if(source.toString().contentEquals("\n"))
		      {
		    	  Toast.makeText(MainActivity.this, "表名不允许出现回车", Toast.LENGTH_SHORT).show();
		    	  return "";
		      }
		      else return null;
		   }
		};
	
	/*
	 * 重写监听事件，为建表等按钮添加监听事件
	 */
	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.newTableButton:
			if(!creatTableText.getText().toString().isEmpty()) {   //如果表名非空，则建表，否则提示
			//获取sharedPreferences中当前数据库版本号，并加1赋值给MySQLite，这样才能保证MySQLite会执行onUpgrade
			int version=sPreferences.getInt("MySQLiteVersion",0);
			version++;
			mSqLite=new MySQLite(this, "QiuRuibo_SQLite", null
					,version);
			Log.i("MainVersion",""+version);
			sData.set(creatTableText.getText().toString());
			mSqLite.getWritableDatabase();
			//将数据库当前版本号存放到sharedPreferences中
			editor.putInt("MySQLiteVersion",version);
			editor.apply();
			}
			else {
				Toast.makeText(this, "表名不能为空", Toast.LENGTH_SHORT).show();
			}
			break;
		case R.id.nextButton:
			Intent intent=new Intent(MainActivity.this,InsertDataActivity.class);
			startActivity(intent);
			onPause();
			break;
		default:
			break;
		}
	}
}
      我们可以注意到，上面的MainActivity并不是直接继承于Activity，而是继承于自建的BaseActivity，为什么要建立这个activity呢？其实作用很简单，建立BaseActivity，可以减少很多activity的重复性操作，本文中，在baseactivity中定义了一个activityCollector管理器，实现每个activity创建后都在List<Activity>里面注册，销毁时都在List<Activity>中注销。这么做有什么用呢？当我们想要一键关闭所有的activity时就显得十分地方便，在想要关闭所有的activity时我们可以在收集activity的管理器中通过一个循环关闭所有的activity。当然建立这么一个父activity还可以实现很多其他的功能，能够避免我们代码中出现大量的重复代码。下面是BaseActivity的代码：

package com.example.sqlite;
 
import android.app.Activity;
import android.os.Bundle;
 
public class BaseActivity extends Activity{
 
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		ActivityCollector.addAvtivity(this);
	}
	@Override
	public void onDestroy() {
		super.onDestroy();
		ActivityCollector.removeActivity(this);
	}
}
     ActivityCollector的代码 

 
public class ActivityCollector {
 
	//创建一个activity的list来收集每一个activity
	public static List<Activity> activities=new ArrayList<Activity>();
	/**
	 * @param 新增传递进来的activity
	 */
	public static void  addAvtivity(Activity activity) {
		activities.add(activity);
	}
	/**
	 * @param 删除传递进来的activity
	 */
	public static void removeActivity(Activity activity) {
		activities.remove(activity);
	}
	/**
	 * @param 删除所有的activity
	 */
	public static void finishAll() {
		for(Activity a:activities)
			if(!a.isFinishing())
				a.finish();
		activities.clear();
	}
}
4、SQLite数据库的增删查改
      在前面建立数据库时我们贴了一大段的代码，里面用于建立数据库的其实只有onCreat方法，那么里面的更新方法是用来干嘛的呢？现在我们就要用到它了，因为一个数据库一旦建立了，那么就不会再执行onCreate方法了，那么如果我们想要新增或者修改一个表的结构怎么办呢，这时就要用到上面的onUpgrade方法了。而onUpgrade方法也是有一定的执行时机的，只有当数据库版本比之前高时才会执行onUpgrade方法，因此如果我们想要修改数据库中表的结构，我们必须在修改时给数据库一个更高的版本，这样才会执行onUpgrade方法。

    我们在编写数据库应用软件时，需要考虑这样的问题：因为我们开发的软件可能会安装在很多用户的手机上，如果应用使用到了SQLite数据库，我们必须在用户初次使用软件时创建出应用使用到的数据库表结构及添加一些初始化记录，另外在软件升级的时候，也需要对数据表结构进行更新。那么，我们如何才能实现在用户初次使用或升级软件时自动在用户的手机上创建出应用需要的数据库表呢？总不能让我们在每个需要安装此软件的手机上通过手工方式创建数据库表吧？因为这种需求是每个数据库应用都要面临的，所以在Android系统，为我们提供了一个名为SQLiteOpenHelper的抽象类，必须继承它才能使用，它是通过对数据库版本进行管理来实现前面提出的需求。 
为了实现对数据库版本进行管理，SQLiteOpenHelper类提供了两个重要的方法，分别是onCreate(SQLiteDatabase db)和onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)，前者用于初次使用软件时生成数据库表，后者用于升级软件时更新数据库表结构。当调用SQLiteOpenHelper的getWritableDatabase()或者getReadableDatabase()方法获取用于操作数据库的SQLiteDatabase实例的时候，如果数据库不存在，Android系统会自动生成一个数据库，接着调用onCreate()方法，onCreate()方法在初次生成数据库时才会被调用，在onCreate()方法里可以生成数据库表结构及添加一些应用使用到的初始化数据。onUpgrade()方法在数据库的版本发生变化时会被调用，一般在软件升级时才需改变版本号，而数据库的版本是由程序员控制的，假设数据库现在的版本是1，由于业务的变更，修改了数据库表结构，这时候就需要升级软件，升级软件时希望更新用户手机里的数据库表结构，为了实现这一目的，可以把原来的数据库版本设置为2(有同学问设置为3行不行？当然可以，如果你愿意，设置为100也行)，并且在onUpgrade()方法里面实现表结构的更新。当软件的版本升级次数比较多，这时在onUpgrade()方法里面可以根据原版号和目标版本号进行判断，然后作出相应的表结构及数据更新。

getWritableDatabase()和getReadableDatabase()方法都可以获取一个用于操作数据库的SQLiteDatabase实例。但getWritableDatabase() 方法以读写方式打开数据库，一旦数据库的磁盘空间满了，数据库就只能读而不能写，倘若使用getWritableDatabase()打开数据库就会出错。getReadableDatabase()方法先以读写方式打开数据库，如果数据库的磁盘空间满了，就会打开失败，当打开失败后会继续尝试以只读方式打开数据库。

     而SQLite数据库的添加数据其实可以有多种方法添加，如果你十分熟悉SQL语言，那么完全可以通过SQL语言来写，最后通过自定义的类MYSQL执行exec。当然安卓也提供了这么一个方法，那就是insert方法，该方法要求需要新增的数据封装到ContentValue里面，然后再进行添加。表的创建以及数据的新增如下：

public class InsertDataActivity extends BaseActivity implements OnClickListener{
	private Button addDataButton,backButton,updateButton;
	private EditText userName,userPassword,verification,chooseTable;
	private SharedPreferences sPreferences;
	private MySQLite mSQLite;
	private AlertDialog alertDialog;
	private ShareData sData;
	private SharedPreferences.Editor editor;
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.insert_data);
		//初始化控件
		addDataButton=(Button)findViewById(R.id.addDataButton);
		backButton=(Button)findViewById(R.id.back);
		updateButton=(Button)findViewById(R.id.updateButton1);
		userName=(EditText)findViewById(R.id.userNameText);
		userPassword=(EditText)findViewById(R.id.userPasswordText);
		verification=(EditText)findViewById(R.id.verificationCodeText);
		chooseTable=(EditText)findViewById(R.id.chooseTableText);
		//设置按钮监听
		addDataButton.setOnClickListener(this);
		updateButton.setOnClickListener(this);
		backButton.setOnClickListener(this);
		sPreferences=getSharedPreferences("MySQLiteVersion", MODE_PRIVATE);
		editor=sPreferences.edit();
		sData=new ShareData();
	}
	@Override
	protected void onResume() {
		// TODO Auto-generated method stub
		super.onResume();
		//从SharedPrefernces获取数据库版本号
				int version=sPreferences.getInt("MySQLiteVersion", 0);
				//创建数据库实例
				if(version==0) {
					alertDialog=new AlertDialog.Builder(InsertDataActivity.this).create();
					alertDialog.setTitle("警告：");
					alertDialog.setMessage("系统检测到数据库中还没有任何表，是否返回创建一张新表");
					alertDialog.setButton(DialogInterface.BUTTON_POSITIVE,"返回", 
							new DialogInterface.OnClickListener() {			
						@Override
						public void onClick(DialogInterface dialog, int which) {	
							finish();
								}
							});
					alertDialog.setButton(DialogInterface.BUTTON_NEGATIVE,"退出", 
							new DialogInterface.OnClickListener() {
								@Override
								public void onClick(DialogInterface dialog, int which) {
									// TODO Auto-generated method stub
									ActivityCollector.finishAll();
								}
							});
					//开启新线程，来进行延时操作
					new Thread(new Runnable() {
						
						@Override
						public void run() {
							try {
								Thread.sleep(900);
								Looper.prepare();
								alertDialog.show();	
								Looper.loop();
							} catch (InterruptedException e) {
								// TODO Auto-generated catch block
								e.printStackTrace();
							}	
						}
					}).start();		
				}
				else {
					Log.i("Version",""+version);
				mSQLite=new MySQLite(this, "QiuRuibo_SQLite", null,version);
				}
	}
	/*
	 * 重写监听方法，实现按钮监听
	 */
	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.addDataButton:
			if(chooseTable.getText().toString().isEmpty())
				Toast.makeText(this, "选择保存的表名不能为空", Toast.LENGTH_SHORT).show();
		    else if(userName.getText().toString().isEmpty())
				Toast.makeText(this, "用户名不能为空", Toast.LENGTH_SHORT).show();
			else if(userPassword.getText().toString().isEmpty())
				Toast.makeText(this, "密码不能为空", Toast.LENGTH_SHORT).show();
			else if(verification.getText().toString().isEmpty())
				Toast.makeText(this, "验证码不能为空", Toast.LENGTH_SHORT).show();
			else {
			add();
			
			}
			break;
		case R.id.back:
			Intent intent=new Intent(InsertDataActivity.this,MainActivity.class);
			startActivity(intent);
			finish();
			break;
		case R.id.updateButton1:
			Intent intent1=new Intent(InsertDataActivity.this,UpdateActivity.class);
			startActivity(intent1);
			finish();
			break;
		default:
			break;
		}
	}
	private void add() {
		// TODO Auto-generated method stub
		SQLiteDatabase sqLiteDatabase=mSQLite.getWritableDatabase();
		ContentValues cValues=new ContentValues();
		cValues.put("username", userName.getText().toString());
		cValues.put("password", userPassword.getText().toString());
		cValues.put("verification", verification.getText().toString());
		 /*创建表，并判断是否已经存在此表，没创建，则创建并初始化*/
        if (!mSQLite.tabIsExist(chooseTable.getText().toString())) {
        	createAlertDialog();
            }
        else {
        	
        	// 开启事务
        	sqLiteDatabase.beginTransaction();
            try{
                    // 数据库操作
            	// 插入数据
            sqLiteDatabase.insert(chooseTable.getText().toString(), null, cValues);
            // 设置事务标志为成功，当结束事务时就会提交事务
            //（一定要记得添加这句，否则就会出现insert成功，但数据库没数据）
            sqLiteDatabase.setTransactionSuccessful();
            Toast.makeText(InsertDataActivity.this, "添加成功", Toast.LENGTH_SHORT).show();
            //将此时的表名传递给更新的activity
            ShareData.setTable(chooseTable.getText().toString());
            }
            catch(Exception e){
            	Toast.makeText(InsertDataActivity.this,
            			e.toString(), Toast.LENGTH_SHORT).show();
             
            } finally {
                    // 结束事务方法时会检查事务的标志是否为成功，
                    // 如果程序执行此方法之前调用了setTransactionSuccessful()方法设置事务的标志为成功则提交事务
              // 如果没有调用setTransactionSuccessful() 方法则回滚事务。
            	sqLiteDatabase.endTransaction();
            }
        }
		cValues.clear();
	}
	/*
	 * 创建一个对话框，用来询问是否自动建表
	 */
	public void createAlertDialog() {
		AlertDialog aDialog=new AlertDialog.Builder(InsertDataActivity.this).create();
		aDialog.setTitle("温馨提示");
		aDialog.setMessage("您想要保存的表名不存在，是否在数据库中自动创建该表？");
		aDialog.setButton(AlertDialog.BUTTON_POSITIVE,"是", 
				new DialogInterface.OnClickListener() {
			@Override
			public void onClick(DialogInterface dialog, int which) {
				//获取sharedPreferences中当前数据库版本号，并加1赋值给MySQLite，这样才能保证MySQLite会执行onUpgrade
				int version=sPreferences.getInt("MySQLiteVersion",0);
				mSQLite=new MySQLite(InsertDataActivity.this, "QiuRuibo_SQLite", null
						,++version);
				Log.i("MainVersion",""+version);
				sData.set(chooseTable.getText().toString());
				mSQLite.getWritableDatabase();
				//将数据库当前版本号存放到sharedPreferences中
				editor.putInt("MySQLiteVersion",version);
				editor.apply();
				
			}
		});
		aDialog.setButton(AlertDialog.BUTTON_NEGATIVE,"否", 
				new DialogInterface.OnClickListener() {
			@Override
			public void onClick(DialogInterface dialog, int which) {
			}
		});
		aDialog.show();
	}
}
        上面的数据的添加是通过安卓提供的API来实现的，下面我们就用SQL语言来实现一个数据的更新的操作。

public class UpdateActivity extends BaseActivity implements OnClickListener{
 
	private SQLiteDatabase db;
	private MySQLite mySQLite;
	private Button updateButton,backButton,nextButton;
	private SharedPreferences sPreferences;
	private EditText updatePasswordText,updateAgainText,usernameToUpdate;
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.update);
		//初始化控件
		updateButton=(Button)findViewById(R.id.updateButton);
		backButton=(Button)findViewById(R.id.back1);
		nextButton=(Button)findViewById(R.id.nextButton1);
		updatePasswordText=(EditText)findViewById(R.id.updatePasswordText);
		updateAgainText=(EditText)findViewById(R.id.updateAgainText);
		usernameToUpdate=(EditText)findViewById(R.id.usernameToUpdatePassword);
		//设置按钮监听
		updateButton.setOnClickListener(this);
		backButton.setOnClickListener(this);
		nextButton.setOnClickListener(this);
		sPreferences=getSharedPreferences("MySQLiteVersion", MODE_PRIVATE);
		int version=sPreferences.getInt("MySQLiteVersion", 0);
		mySQLite=new MySQLite(this, "QiuRuibo_SQLite", null,version);
		
		
	}
	/*
	 * 重写方法，实现按钮监听
	 */
	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.back1:
			Intent intent=new Intent(UpdateActivity.this,InsertDataActivity.class);
			startActivity(intent);
			finish();
			break;
		case R.id.updateButton:
			if(updatePasswordText.getText().toString().equals
					(updateAgainText.getText().toString()))
			update();
			else {
				Toast.makeText(this, "两次输入的密码不一致", Toast.LENGTH_SHORT).show();
			}
			break;
		case R.id.nextButton1:
			
			break;
		default:
			break;
		}
	}
	private void update() {
		Cursor cursor=null;
		boolean isExist=false;
		db=mySQLite.getReadableDatabase();
		cursor=db.rawQuery("select count(username)  from "+ShareData.getTable()
		                +" where username='"+usernameToUpdate.getText().toString()
		                +"' ", null);
		 if(cursor.moveToNext()){
             int count = cursor.getInt(0);
             if(count>0){
                     isExist=true;
             }
     }
		 if(isExist) {
		db.execSQL("update "+ShareData.getTable()+" set password='"
	                    +updatePasswordText.getText().toString().trim()+"' where username='"
				        +usernameToUpdate.getText().toString().trim()+"'");
		Toast.makeText(UpdateActivity.this, "修改成功", Toast.LENGTH_SHORT).show();
		 }
		 else 
			 Toast.makeText(UpdateActivity.this, "用户名不存在", Toast.LENGTH_SHORT).show();
		
	}
}
        两种方法各有优缺点，看自己的知识掌握选择，当然我们还可以使用一些开源项目来进行数据库的操作，像LitePal等，LitePal是一种对象关系映射（ORM）的模式，可以帮助我们以面向对象的语言来实现数据库的操作，对于不熟悉SQL语言的人无疑是一个很大的福利。LitePal的操作这里就不介绍了，感兴趣的可以自己上Github下载使用，当然使用Android studio的就不用那么麻烦，可以直接新建一个xml文件就能够成功配置。

        补充一点：上面我们一直使用一个自定义类ShareData来实现不同类或者activity之间的数据共享，该类定义如下：

public class ShareData {
 
	private static String table="defaultTable",tableFromInsert;
	//private Context context;
	public ShareData(){
		//this.context=context;
	}
	public static void setTable(String table) {
		tableFromInsert=table;
	}
	public static String getTable() {
		return tableFromInsert;
	}
	public void set(String tableName) {
		table=tableName;
	}
	public String get() {
		return table;
	}
}
