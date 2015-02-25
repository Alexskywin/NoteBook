#Laravel 5模版的值绑定方式

以下内容都放在对应Controller的Action中.  
假设在PagesController的about中, 模版文件放在resources/views/pages/about.blade.php中

##方法一

	class PagesController extends Controller {
	    public function about()
	    {
	    	$name = 'Summer.夏天';
	    	
	    	return view('pages.about')->with('name', $name);
	    }
	}
	
##方法二
	class PagesController extends Controller {
	    public function about()
	    {
	    	$name = 'Summer.夏天';
	    	
	    	//这里使用的是魔术方法
	    	return view('pages.about')->withName($name);
	    }
	}
	
##方法三
	class PagesController extends Controller {
	    public function about()
	    {
	    	$data = [];
	    	$data['first'] = 'Summer.';
	    	$data['last']  = '夏天';
	    	
	    	return view('pages.about')->with($data);
	    }
	}
	
##方法四
	class PagesController extends Controller {
	    public function about()
	    {
	    	$data = [];
	    	$data['first'] = 'Summer.';
	    	$data['last']  = '夏天';
	    	
	    	//view快捷方法的第二个参数是这么用的.
	    	return view('pages.about', $data);
	    }
	}

##方法五
	class PagesController extends Controller {
	    public function about()
	    {
	    	$first = 'Summer.';
	    	$last  = '夏天';
	    	
	    	//稍微变动下
	    	return view('pages.about', compact('first', 'last'));
	    }
	}
	
我个人比较喜欢方法三和方法四, 在模板中读取的方式其实很简单:

假设模版about.blade.php读取变量为$name = 'Summer.夏天';

{{ $name }} => 输出 "Summer.夏天"  
{!! $name !!} => 输出 "Summer.夏天"

两个方式不同点在于, {{ $name }} 会将html标签转义, {!! $name !!}会将html原样输出.