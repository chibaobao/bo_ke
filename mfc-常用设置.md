---
title: mfc-常用设置
date: 2016-10-10 19:33:03
tags: mfc
categories: 编程学习
---

关于mfc的各种控件类的基本设置使用
# MFC-2

## CDialog

- DOModal() //模态对话框
	1. 资源视图添加对话框资源
	2. 给资源关联类（右键添加类）
		//enum Id=资源id
	3. 包含头文件
	4. 创建对象，并.DoModal()

- ShowWindow() //非模态对话框
	 1. 创建模态对话框的1，2，3步
	 2. 创建对象(定义为类的成员变量就不会一闪而过)
	 3. 初始化对象：.Create(资源ID,parent)//parent默认参数空，在类的构造函数里面
	 4. ShowWindow()
- OnInitDialog()
	-  作用：控件的初始化
	-  调用时机：窗口被初始化完成而且还没显示
	-  注意：不能在构造函数中初始化
- ONPaint()
	- WM_PAINT对应，窗口刷新时被调用
- 创建对话框

## 常用控件和类
### 类
- CString
	- 字符串拼接

	```cpp
	CString info;
	info.Format(_T("用户名: %s, 密码: %s"), m_user, m_passwd);
	```

### 控件

- CStatic 静态控件
	- 文本显示，
	- 图片显示，将ID-->IDC_STATIC更改

	```cpp
		//管理变量(假设关联了了m_pic变量)
		m_pic.ModifyStyle(0,SS_BITMAP | SS_CENTERIMAGE)
		m_bitmap.LoadBitmapw(IDB_BITMAP1);//m_bitmap是CBitmap类型，类成员变量否则一闪而过，IDB_BITMAP1类资源视图的图片的ID名
		m_pic.SetBitmap((HBITMAP)m_bitmap);
	```
- CBUtton 按钮

- CEdit编辑框控件
	- 文本输入框
	- 常用属性
		- Number
		- 属性设置

| 属性 | 含义 |	
| -------- | --------------------------------------------------- |	
|Number				|	True只能输入数字|
|Password			|	True密码模式
|Want return	 	| True接收回车键，自动换行，只有在多行模式下，才能换行
|Multiline			| True多行模式
|Auto VScroll 		|	True 当垂直方向字符太多，自动出现滚动条，同时设置Vertical Scroll才有效
|Vertical Scroll 	|	True当垂直方向字符太多，自动出现滚动条，和Auto VScroll配合使用
|Horizontal Scroll---|True当垂直方向字符太多，自动出现滚动条，和Auto HScroll配合使用
|Read Only			| True 只读

- CComboBox下拉框控件
	- 属性设置
		- Date    aa;bb;cc //初始化
		- Type  下拉框样式
		- Sort    进行排序
	- 常用接口

| 接口 | 功能 |
    |-----------|---------------|
	|CComboBox::AddString|组合框添加一个字符串
    |CComboBox::SetCurSel	     |设置当前选择项(当前显示第几项)，下标从0开始|
    |CComboBox::GetCurSel    	 |获取组合框中当前选中项的下标|
    |CComboBox::GetLBText	     |获取指定位置的内容|
    |CComboBox::DeleteString	 |删除指定位置的字符串|
    |CComboBox::InsertString	 |在指定位置插入字符串 |

```cpp
		示例
		获取当前字符串并显示
		{
			CString text;
			int index = m_combox.GetCurSel();
			m_combox.GetLBText(index, text); 
			// MessageBox的方式弹框显示
			MessageBox(text);
		}
		
		删除当前选中的项
		{
			// 获取当前选中的item的下标
			int index = m_combox.GetCurSel();
			// 删除index对应的字符串
			m_combox.DeleteString(index);
			// 默认显示第一个
			m_combox.SetCurSel(0);
		}
		
		插入项
		{
			// 当前位置
			int index = m_combox.GetCurSel();
			// 获取编辑框中的数据 -- m_str，m_str是与一个编辑框关联的变量
			UpdateData(TRUE);
			// 将字符串插入到index的位置
			m_combox.InsertString(index, m_str);
			// 显示插入的字符串
			m_combox.SetCurSel(index);
		}
```
- CListCtrl列表控件

	- 常用接口
	
|接口	|功能|
	|--------|------|
	|CListCtrl::SetExtendedStyle	|设置列表风格|
	|CListCtrl::SetExtendedStyle	|获取列表风格|
	|CListCtrl::InsertColumn	        |插入某列内容，主要用于设置标题|
	|CListCtrl::InsertItem	            |在某行插入新项内容|
	|CListCtrl::SetItemText	            |设置某行某列的子项内容|
	|CListCtrl::GetItemText	        |获取某行某列的内容|

	- 常用属性
		- view -> Report(设置报表方式显示)

```cpp
	//m_list与CListCtrl关联的变量
	// CListCtrl属性设置: 加网格, 整行选中
	m_list.SetExtendedStyle(LVS_EX_GRIDLINES | LVS_EX_FULLROWSELECT |
		m_list.GetExtendedStyle());
	// 表头的初始化操作
	CString name[] = {
		_T("姓名"),
		_T("性别"),
		_T("年龄")
	};
	for (int i = 0; i < sizeof(name) / sizeof(name[0]); ++i)
	{
		// 插入列
		m_list.InsertColumn(i, name[i], LVCFMT_LEFT, 150);
	}
	// 添加20行
	for (int i = 0; i < 20; ++i)
	{
		CString str;
		// 添加一个新的行, 并初始化第一列 0列
		str.Format(_T("小龙女_%d"), i);
		m_list.InsertItem(i, str);
		// 初始化第1列
		m_list.SetItemText(i, 1, _T("女"));
		// 初始化第2列
		// 数据格式化
		str.Format(_T("%d"), i + 20);
		m_list.SetItemText(i, 2, str);

	}
	```
- CTreeCtrl 树控件

```
	// 初始化树控件
	//根节点
	CString roots[] = {
		_T("北京"),
		_T("上海"),
		_T("广州")
	};
	// 初始化图片列表
	m_image.Create(32, 32, ILC_COLOR32, 4, 4);
	// 向图片列表中添加图片
	// 在应用程序类中有这样一个函数, 可以加载图标
	// 资源id就是 int 类型
	int ids[] = { IDI_ICON1, IDI_ICON2, IDI_ICON3, IDI_ICON4 };
	for (int i = 0; i < 4; ++i)
	{
		m_image.Add(AfxGetApp()->LoadIconW(ids[i]));
	}
	// 添加图片
	m_tree.SetImageList(&m_image, TVSIL_NORMAL);
	// 创建节点数组, 保存根节点句柄
	HTREEITEM rootItems[3];
	// 初始化根节点
	for (int i = 0; i < 3; ++i)
	{
		rootItems[i] = m_tree.InsertItem(roots[i], i, i+1);
		if (i == 0)
		{
			m_tree.InsertItem(_T("昌平"), i, i+1, rootItems[i]);
			m_tree.InsertItem(_T("海淀"), i, i + 1, rootItems[i]);
		}
		else if (i == 1)
		{
			m_tree.InsertItem(_T("黄埔"), i, i + 1, rootItems[i]);
			m_tree.InsertItem(_T("青浦"), i, i + 1, rootItems[i]);
		}
		else if (i == 2)
		{
			m_tree.InsertItem(_T("白云"), i, i + 1, rootItems[i]);
			m_tree.InsertItem(_T("长河"), i, i + 1, rootItems[i]);
		}
	```
- CTabSheet

- CTabCtrl--Tab页窗口
	- 用封装好的CTabSheet类
	- 步骤:
	1. 在ui编辑界面箱拖放 Tab Control控件
	2. 把 TabSheet.h和TabSheet.cpp添加到当前项目中
	3. 给Tab Control控件 关联Control类别变量类型为CTabSheet的变量
	4. 添加对话框资源
		1. 资源视图 -> Dialog -> 右击 -> 插入 Dialog
		2. 添加对话框资源
		Style -> Child (子窗口)
		Border -> None (无边框)
		3. 给对话框资源关联类：选择对话框模板 -> 右击 -> 添加 (如:MyDlg1)MyDlg1dlg1;
		4. 主对话框类中, 定义自定义类对象，需要相应头文件
		5. 主对话框类中 OnInitDialog() 做初始化工作
### 常用控件使用
- 在程序中对控件进行处理要 ***添加变量***
	- Contol类别
	  相当于将操作变量就操作控件
	- Value类别
		- 刷新变量与控件 
		Update(TRUE)    控件到变量
		Update(FALSE)    变量到控件
- 默认不处理事件的控件，将ID-->IDC_STATIC更改

