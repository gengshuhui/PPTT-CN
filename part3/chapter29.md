# 29	将事件绑定命令
本章介绍Tk中的事件绑定机制。像鼠标点击或击键一样，绑定将事件关联一个Tcl命令。当然也有定义虚拟事件的机制，如`<<Cut>>`和`<<Paste>>`事件在不同的平台上关联到不同的击键上。这里讨论的Tcl命令有：bind、bindtags和event。  
绑定将Windows系统的一系列事件关联到一个Tcl命令上。事件包括按键、键释放、按按钮、释放按钮、鼠标进入窗口、鼠标离开、窗口尺寸改变、窗口打开、窗口关闭、得到焦点、失去焦点、控件销毁。绑定定义在绑定标签上，且每一个控件关联到一个有序绑定标签集。绑定标签在绑定和控件间提供了一个级别的迂回，创建了一个灵活强大的系统。（The binding tags provide a level of indirection between bindings and widgets that creates a flexible and powerful system.）  
虚拟事件用来在不同的平台上支持不同的观感。像`<<Copy>>`这样的虚拟事件是一个高级别的名字，像`<Control-c>`和`<Key-F6>`是低级别的事件名字。虚拟事件隐藏了不同平台上相同逻辑操作的不同键击差别。Tk定义了新的虚拟事件，且应用程序可以定义其自己的虚拟事件。## 29.1	bind命令
bind命令创建事件绑定，并返回当前绑定的信息。命令的一般形式为：
```bind bindingTag ?eventSequence? ?command?
```
如果所有参数都存在，从eventSequence到command的绑定是为bindingTag而定义的。bindingTag是控件类名（如，Button）或是控件实例名（如，.button.foo）。绑定标签后面会有详述。bind以单个参数调用时，即一个绑定标签，返回有命令绑定的事件。
```bind Menubutton=> <Key-Return> <Key-space> <ButtonRelease-1>    <B1-Motion> <Motion> <Button-1> <Leave> <Enter>```本例中的事件是按键和鼠标移动。`<Button-1>`是当用户按下第一个或鼠标左键时产生的事件。`<B1-Motion>`是当按下第一个鼠标键用户移动鼠标时产生的事件。
`<Key-space>`事件时用户按空格键时产生的。单角括号限定了单个事件，且你可以为事件序列定义绑定。事件语法在第439页中描述，事件序列在第445而中描述。  
如果bind后跟一个绑定标签和一个事件序列，它将返回绑定到此事件序列的Tcl命令：
```bind Menubutton <B1-Motion>=> tk::MbMotion %W down %X %Y```
事件绑定中的TCL命令为事件关键字支持额外的语法。这些关键字以百分号开始有一个或多个标识事件某一属性的字符。在TCL命令执行前这些关键字会被替换为与特定事件相关的数据。如，%W被控件的路径名替换。%X和%Y关键字会被事件的屏幕相关坐标所替换。%x和%y关键字会被控件的屏幕相关坐标所替换。事件关键字在第448页中有总结。%替换会在绑定到事件的整个命令中执行，不管其它的引用机制。你必须用%%来得到单个百分号。鉴于此你应该使绑定命令尽量短，如果必要的话就增加一个新的过程（如，tk::MbMotion），而不是在整个代码中乱丢百分号。 
 创建一个新的绑定通过指定一个绑定标签、一个事件序列和一个命令来实现：
```bind Menubutton <B1-Motion> {tk::MbMotion %W down %X %Y}```如果绑定命令的第一个字符是+，命令（不带+）就被加到为那个事件和绑定标签的命令（如果有的话）中。   
```bind bindingTag event {+ command args}```
要删除事件的绑定，绑定事件到空字符串上：
```bind bindingTag event {}```**绑定在全局范围内执行。**  
当绑定触发时，命令在全局范围内执行。常见的错误是混淆了bind命令创建绑定时活动的范围与绑定触发时活动的范围。相同的问题也会突然出现在与按钮相关联的命令上，后文在在第30章开头会详细讨论。
## 29.2	bindtags命令
绑定标签将相关绑定分组，每一个控件都与一个有序绑定标签集相关联。控件和和绑定间的The level of indirection使你在绑定标签上将功能分组并且由不同的绑定标签组成控件行为。（The level of indirection between widgets and bindings lets you group functionality on binding tags and compose widget behavior from different binding tags.）  例如， all绑定标签在`<Tab>`上有绑定，它在控件间改变焦点。Text绑定标签在击键上有绑定，它插入并编辑文件。只有文本控件使用Text绑定标签，但所有控件共享all 绑定标签。你可以引入新的绑定标签并动态改变控件和绑定标签的关联。结果是强大灵活的管理绑定的方式。 
	 bindtags命令设置或查询控件的绑定标签。bindtags命令的一般形式为：
```bindtags widget ?tagList?
```
下面的命令返回文本控件.t的绑定标签：
```bindtags .t=> .t Text . all
```
你可以改变绑定标签和其顺序。bindtags的tagList参数必须是一个正确的TCL列表。下面的命令重新排列.t的绑定标签顺序并删除.绑定标签。
```bindtags .t [list all Text .t]```
默认情况下，除了顶层窗口所有的Tk控件都以下列顺序有4个绑定标签：
* 控件的Tk路径名（如，.t）。使用这个绑定给特定的控件提供特殊的行为。默认情况下在这个绑定标签上没有绑定。* 控件的类（如，Text）。控件的类源于创建它的命令的名字。按钮控件类名为Button，文本控件类名为Text，等等。Tk控件在它们的类上用绑定定义了默认行为。* 控件顶层窗口（如，.）的Tk路径名。这对于顶层窗口来说是多余的，所以它不会使用两次。默认在这个绑定标签上没有绑定。顶层窗口上的绑定可以在对话框中用来处理加速键。* 全局绑定标签all。在all上的绑定用来在控件间改变焦点。在第604页中进行讨论。当控件上有超过一个绑定标签的时候，则每一个绑定标签上的一个绑定都匹配一个事件。绑定以绑定标签的顺序处理。默认情况下，最精确的标签先来，最一般的绑定标签后来。  
例29-1有两个有下列行为的框架控件。当鼠标进入它们时，它们变红。当鼠标离开时，它们变白。当用户键入`<Control-c>`时，鼠标下面的框架被销毁。其中一个控件，.tow报告鼠标当前点击的坐标。
	### 29.2.1	例29-1不同绑定标签上的绑定
```frame .one -width 30 -height 30frame .two -width 30 -height 30bind Frame <Enter> {%W config -bg red}bind Frame <Leave> {%W config -bg white}bind .two <Button> {puts "Button %b at %x %y"}pack .one .two -side leftbind all <Control-c> {destroy %W}bind all <Enter> {focus %W}```
Frame类在`<Enter>`和`<Leave>`上有一个绑定，当鼠标进入和离开窗口时改变框架的背景颜色。这个绑定共享给所有的框架。在all上也有一个`<Enter>`的绑定，这个绑定设置键盘焦点。当鼠标进入一个框架时两个绑定都被触发。### 29.2.2	焦点和按键事件
在`<Control-c>`上的绑定被所有控件共享。此绑定销毁目标控件。因为这是一个键击，所以得到相应控件上的键盘焦点是很重要的。默认情况下，焦点在主窗口上，销毁它就终止了整个应用程序。当你将鼠标移到控件上面的时候，`<Enter>`的全局绑定就给控件以焦点。在本例中，将鼠标移到控件之上然后按`<Control-c>`会销毁这个控件。如果你喜欢点击式的焦点模式的话就将focus命令绑定到`<Button>`上，而非`<Enter>`。焦点在在第39章中描述。
### 29.2.3	在绑定中使用break和continue
break和continue命令控制绑定标签集中的绑定的相继性（progession）。break命令停止当前的绑定并抑制绑定标签集顺序中余下的绑定标签。在一个绑定中的continue命令停止当前的绑定并从下一个绑定标签开始以此命令继续。  
比如，Entry控件绑定标签有在单行entry控件中插入和编辑文本的绑定。你可以将绑定放在`<Return>`上，它使用控件的值执行TCL命令，下面的例子中在\r字符被加到entry控件前运行Some Command。绑定就在控件的名字上，它是绑定标签集的第一个，所以break抑制了Entry插入字符的绑定：
```bind .entry <Return> {Some Command ; break}
```注意，你不能在被绑定调用的过程中使用使用break或conitnue命令。这是因为过程机制不会传播break或continue信号。相反，你可以使用-code选项来返回，这在第86页中描述：
```return -code break
```### 29.2.4	定义新的绑定标签
你可以通过只使用bind或bindtags命令来引入新的绑定标签。绑定标签在将绑定分成不同的集合时很有用，如一个编辑器的不同模式的专门绑定。比如，仿效vi编辑器就是使用两个绑定标签，一个是插入模式另一个是命令模式。用户键入i时进入插入模式，键盘入`<Escape>`进入命令模式：
```bindtags $t [list ViInsert Text $t all]bind ViInsert <Escape> {bindtags %W {ViCmd %W all}}bind ViCmd <Key-i> {bindtags %W {ViInsert Text %W all}}
```Text类绑定用于插入模式。将控件置入命令模式的命令是放上一个新的绑定标签，ViInsert，而不是改变Text的默认绑定。bindtags命令通过改变控件的绑定标签集来改变模式。%W被会控件的名字所代替，本例中它与$t是一样的。当然，要完全实现所有的vi命令需要定义更多的绑定。## 29.3	事件语法
bind命令使用下面的语法来描述事件：
```<modifier-modifier-type-detail><<Event>>```
第一种形式适用于像键击和鼠标移动这样的物理事件。第二种形式适用于像剪切和粘贴这样的虚拟事件，它在不同的平台上对应着不同物理事件。物理事件在本章中描述。虚拟事件在第446而中详述。  
此描述中主要的部分是type（如，Button或Motion）。为了识别按键或按钮（如，Key-a或Button-1）在一些事件中使用detail。modifier是事件发生时（如，Control-Key-a或B2-Motion）已经按下的另一个按键或按钮。可以有多重修饰成分（如，Control-Shift-x）。<and>限定为单一事件。  
表29-1列出了所有的事件类型。当两个事件一起列出时（如，ButtonPress和Button）它们是等价的。
Table 29-1. 事件类型
command      		| explain
-------------------| --------------------Activate		|	应用程序已经被激活。 (Macintosh)ButtonPress, Button		|	按钮按下ButtonRelease	|	按钮释放。Circulate		|	窗口的堆叠顺序改变了。CirculateRequest	|	应用程序请求改变其窗口堆叠顺序（由窗口管理器使用）Colormap		|	color map已经改变Configure		|	窗口改变了大小、位置、边界或堆叠顺序。ConfigureRequest	|	应用程序请求改变其窗口配置（由窗口管理器使用）Create			|	应用程序请求创建一个窗口（由窗口管理器使用）Deactivate		|	应用程序已经被去激活。 (Macintosh)Destroy			|	窗口已经被销毁。Enter			|	鼠标已进入窗口。Expose			|	窗口已暴露。FocusIn			|	窗口已得到焦点。FocusOut		|	窗口已失去焦点。Gravity			|	因父窗口大小变化而使此窗口已经移动。KeyPress, Key	|	有键按下。KeyRelease		|	有键释放。Leave			|	鼠标正离开窗口。Map				|	窗口已经被绘制（打开）。MapRequest		|	应用程序请求绘制一个窗口（由窗口管理器使用）Motion			|	鼠标正在窗口中移动。MouseWheel		|	鼠标滚轮已经移动。Property		|	窗口的一个属性（页？）已经改变或删除。Reparent		|	窗口重新被指定父控件。ResizeRequest	|	应用程序请求调节窗口大小（由窗口管理器使用）Unmap			|	窗口已被不绘制（unmapped ）了（图标化）。Visibility		|	窗口变了可见性。
### 29.3.1	键盘事件
KeyPress类型事件与KeyRelease是有区别的，以便你可以对这些每一个事件定义不同的绑定。KeyPress可以简写为Key，且如果详细指定键时Key也可以省略。最终，作为KeyPress事件的一个特例，尖括号也可以省略。下面都是等价的事件描述：
```<KeyPress-a><Key-a><a>a```
按键的细节称为**keysym**，它引用键盘上键的图形打印。对于标点符号和非打印字符，定义了特殊的keysym。在keysym中大小写是非常重要的，但不幸的是还没有统一的方案。特别地，BackSpace有大写字母的B和S。通常遇到的keysym包括：Return, Escape, BackSpace, Tab, Up, Down, Left, Right, comma, period, dollar, asciicircum, numbersign, exclam。从Tk8.3.2开始，联机帮助包括了一个新的有所有标准keysym的keysym参考页。  **找出你的键盘产生什么keysyms。**  
曾经你不知道你的键盘上某个特别的键产生了什么样的keysym。keysyms由windows系统实现所定义，在UNIX系统中它们受动态键盘映射所影响—X modmap。你可能会发现下面的绑定用于确定你的系统上某个键产生什么keysym是非常有用的。
```bind $w <KeyPress> {puts stdout {%%K=%K %%A=%A}}
```关键字%K会被来自事件的keysym所代替。%A会被来自于事件和任何像Shit这样的饰成分所产生的打印字符所替换。%%会被单个百分号代替。请注意，不管用于分组的分括号这些替换总会发生。如果用户键入Q，则会产生两个KeyPress事件，一个是Shift按键，一个是q键。输出是：
```%K=Shift_R %A={}%K=Q %A=Q
```Shift_R keysym表示右边的shift键被按下。当修饰键按下时关键字%A会被{}所代替。如果只有一个修饰键被按下时你可以在<KeyPress>绑定中为此进行检测以免做其它事情。双引号对于强制字符串进行比较是必须的：
```bind $w <KeyPress> {    if {"%A" != "{}"} {%W insert insert %A}}```
### 29.3.2	鼠标事件
ButtonPress(或Button)、ButtonRelease类型的按钮事件是有区别的。如果详细指定一个数字键时Button可以省略。下面都是等价的事件描述
```<ButtonPress-1><Button-1><1>```
注意：事件<1>表示ButtonPress事件，而事件1表示KeyPress事件。为了避免混乱，应该总是指定Key或Button类型。  
通过绑定到Enter、Leave和Motion事件可以跟踪鼠标。当鼠标进入和离开控件时，Enter和Leave分别被触发。当鼠标在控件内移动时产生Motion事件。  
鼠标事件发生时的坐标由绑定命令中的%x和%y关键字表示。坐标是与控件相关联的，原点位于控件窗口的左上角。关键字%X和%Y表示与屏幕相关的坐标：
```	bind $w <Enter>  {puts stdout "Entered %W at %x %y"}bind $w <Leave>  {puts stdout "Left %W at %x %y"}bind $w <Motion> {puts stdout "%W %x %y"}
```鼠标拖拽事件是一个当用户按住一个鼠标按钮时发生的Motion事件。在这种情况下鼠标按钮是一个修饰者，在第443页中会有详细的讨论。绑定类似下面这样：
```bind $w <B1-Motion> {puts stdout "%W %x %y"}
```### 29.3.3	其它事件
当窗口打开和关闭时或控件被几何管理器pack、unpack时会发生<Map>和<Unmap>事件。  
当应用程序被操作系统激活时会产生<Active>和<Deactive>事件。这也适用于Macintosh系统，且当用户点击应用程序主窗口时也会发生。  
当窗口改变大小时会产生`<Configure>`事件。例如，一个基于其大小来计算其显示区域的画布可以绑定一个重显示过程于`<Configure>`事件上。`<Configure>`事件也可以被交互式大小调节所产生。也会由改变控件大小的配置命令产生。当处理`<Configure>`事件时你不能重新配置控件的大小以免产生这些事件的无限次序列。  当窗口被销毁时产生`<Destory>`事件，你也可以中止删除窗口的请求。参看第657页中的wm命令的描述。  
`<MouseWheel>`事件是在Windows中，微软鼠标中的小滚轮产生的。它使用%D关键字报告一个delta值。当前的delta值是一个整数乘以120，其中正值表示向上滚动，负值表示向下滚动。注意，大多数的UNIX系统并不报告`<MouseWheel>`事件，但是一些的确通过`<ButtonPress-4>`和`<ButtonPress-5>`事件报告鼠标滚轮的移动。 
第39章介绍了一些使用`<FocusIn>`和`<FocusOut>`事件的例子。表29-1中的其它事件与X协议中的黑边角有关，且很少使用。关于这些事件可以在Xlib参考手册(Adrian Nye, O'Reilly & Associates, Inc., 1992).中找到更多的信息。
### 29.3.4	顶层窗口的绑定
**顶层窗口的绑定可以被其包含的控件所共享。**
绑定事件到顶层窗口时要小心，因为它们的名字作为一个作用在所有它们所包含的控件上的一个绑定标签。比如，当用户销毁主窗口时，也就是应用程序要退出的时候，下面的绑定会被引发：
```bind . <Destroy> {puts "goodbye"}
```不幸的是，作为一个副作用主窗口中的所有控件都会被销毁，且它们都共享了它们的顶层控件的名字作为一个绑定标签。所以主窗口中的所有控件都销毁时绑定都会被引发。典型地，你只是想做一次某件事情。下面的绑定在执行动作前机查控件的身份：
```bind . <Destroy> {if {"%W" == "."} {puts "goodbye"}}
```## 29.4	修饰成份
修改成份表示事件当时有另一个按钮或按键被按下。典型的修饰成份是Shift和Control键。鼠标按钮也可以作为修饰成份。如果事件不指定任何修改成份，则修饰键的存在会被事件分发者所忽略。然而，如果有两个可能匹配事件发生，则会使用更精确的匹配者。例如，考虑下面三个绑定：
```bind $w <KeyPress> {puts "key=%A"}bind $w <Key-c> {puts "just a c"}bind $w <Control-Key-c> {exit}
```最后一个事件比其它两个更精确。当用户按下Control时又键入c时，第二个绑定会被触发。Meta键会被忽略因为它不匹配任何绑定。如果用户键入的不是c而是其它键，则第一个绑定会被触发。如果用户按下Shift键，则产生的keysym是C而不是c，所以后面两个事件不匹配。
///////////////////////**下面为要点而非全译**/////////////////////
--------
Control、Shift和Lock基本在所有键盘上都可以找到。Meta和Alt可能由于系统不同而不同，常映射到同Mod1或Mod2一样，Tk将会试着确定映射是如何做的。Mod3到Mod5映射到其它特殊键上。
按钮修饰成份B1到B5常用于Motion事件中区分不同的鼠标拖拽操作。如<B1-Motion>为拖拽鼠标左键时产生。
**双击警告。**
Double, Triple和 Quadruple事件在短时内会重复匹配一个事件。译者释：三击可能会顺次匹配到单击、双击、三击事件绑定，与各次点击的时间间隔有关。在字处理软件中会很有用。
```bind . <1> {puts stdout 1}bind . <Double-1> {puts stdout 2}bind . <Triple-1> {puts stdout 3}
```如果想关闭这些，可以使用after来进行延时，<Double>的绑定实现时间常数是500毫秒。
Table 29-2. Event modifiers事件修饰成份
command      		| explain
-------------------| --------------------Control	|	The control key.Shift	|	The shift key.Lock	|	The caps-lock key.Command	|	The command key. (Macintosh)Meta, M	|	Defined to be what ever modifier (M1 through M5) is mapped to the Meta_L and Meta_R keysyms.Alt		|	Defined to be the modifier mapped to Alt_L and Alt_R.Mod1, M1	|	The first modifier.Mod2, M2, Alt	|	The second modifier.Mod3, M3		|	Another modifier.Mod4, M4		|	Another modifier.Mod5, M5	|	Another modifier.Button1, B1	|	The first mouse button (left).鼠标左键Button2, B2	|	The second mouse button (middle).鼠标右键Button3, B3	|	The third mouse button (right).鼠标中键Button4, B4	|	The fourth mouse button. 鼠标第四键Button5, B5	|	The fifth mouse button. 鼠标五键Double	|	Matches double-press event.双击事件Triple	|	Matches triple-press event.三击事件Quadruple	|	Matches quadruple-press event.四击事件Any		|	Matches any combination of modifiers. (Before Tk 4.0)
Unix的xmodmap程序返回键到这些修饰成份的映射。### 29.4.1	例29-2 UNIX的xmodmap程序的输出
```xmodmap: up to 3 keys per modifier,       (keycodes in parentheses):shift Shift_L (0x6a), Shift_R (0x75)lock Caps_Lock (0x7e)control Control_L (0x53)mod1 Meta_L (0x7f), Meta_R (0x81)mod2 Mode_switch (0x14)mod3 Num_Lock (0x69)mod4 Alt_L (0x1a)mod5 F13 (0x20), F18 (0x50), F20 (0x68)```
## 29.5	事件序列
bind后面的事件序列，abc是三个键事件：
```bind . a {puts stdout A}bind . abc {puts stdout C}
```注意在绑定中使用break保证不干其它事：
```bindtags $w [list $w Text [winfo toplevel $w] all]bind $w <Control-x> breakbind $w <Control-x><Control-s> {Save ; break}bind $w <Control-x><Control-c> {Quit ; break}
```emacs中习惯将`<Meta-x>`与`<Escape>x`等价：
### 29.5.1	例29-3 Meta和Escape的类Emacs绑定约定
```proc BindSequence { w seq cmd } {   bind $w $seq $cmd   # Double-bind Meta-key and Escape-key   if [regexp {<Meta-(.*)>} $seq match letter] {      bind $w <Escape><$letter> $cmd   }   # Make leading keystroke harmless   if [regexp {(<.+>)<.+>} $seq match prefix] {      bind $w $prefix break   }}
```   break和continue在Tk3.6及以前版本不支持，所以得绑定为空格（非空，为空的话是删除绑定，空格就是无害化）：
```bind $w $prefix { }```
## 29.6	虚拟事件
一个虚拟事件对应于一个或多个事件序列。任意一个事件序列发生则此虚拟事件就发生了。
### 29.6.1	例29-4 cut、copy和paste的虚拟事件
```switch $tcl_platform(platform) {   "unix" {      event add <<Cut>> <Control-Key-x> <Key-F20>      event add <<Copy>> <Control-Key-c> <Key-F16>      event add <<Paste>> <Control-Key-v> <Key-F18>   }   "windows" {      event add <<Cut>> <Control-Key-x> <Shift-Key-Delete>      event add <<Copy>> <Control-Key-c> <Control-Key-Insert>      event add <<Paste>> <Control-Key-v> <Shift-Key-Insert>   }   "macintosh" {      event add <<Cut>> <Control-Key-x> <Key-F2>      event add <<Copy>> <Control-Key-c> <Key-F3>      event add <<Paste>> <Control-Key-v> <Key-F4>   }}
```可以定义多个映射到相同虚拟事件上的物理事件：
```event add <<Cancel>> <Control-c> <Escape> <Command-period>
```默认虚拟事件定义会添加到相同虚拟事件的已有定义中，下面与上等同：
```event add <<Cancel>> <Control-c>event add <<Cancel>> <Escape>event add <<Cancel>> <Command-period>
```有些控件使用虚拟事件作为通知机制。通过产生虚拟事件来对不同条件作出响应以便你可以创建对这些条件的绑定。如列表控件的<<ListboxSelect>>虚拟事件，最简单的响应方式是绑定虚拟事件：
```bind .lbox <<ListboxSelect>> {ListboxChanged %W}
```## 29.7	产生事件
可以用event generate命令在程序中产生事件，本质上是模拟用户交互。可以产生标准窗口事件或虚拟事件。但只能产生当前应用程序的事件，不能发送事件到其它正在运行的程序中。即不能用此命令控制其它程序。
第一参数是目标窗口，可以是控件的路径名，窗口识别符（winfo id的返回）。  第二参数是事件指定，同创建绑定同。但不能产生事件序列。```event generate .b <ButtonPress-3>
```**控件能接收键事件的条件是必须有焦点。**
聚焦一个控件：
```focus .e1event generate .e1 <KeyPress-a>
```event generate也可以带事件的其它参数，如鼠标的x和y位置。表29-4列出了其选项。需要注意-wrap选项，如果提供True，则鼠标指针会移动到产生事件的x和y位置。event generate . <Motion> -x 10 -y 20 -warp 1## 29.8	事件总结
### 29.8.1	事件命令语法Table 29-3. The event commandevent add virt phys1 phy2 ...	Adds a mapping from one or more physical events to virtual event virt.event delete virt	Deletes virtual event virt.event info	Returns the defined virtual events.event info virt	Returns the physical events that map to virt.event generate win event ?opt val? ...	Generates event for window win. The options are listed in Table 29-4.### 29.8.2	事件关键字注意事件关键字替换会在整个命令中发生而不管TCL的引号机制，所以必须使绑定命令短小，必要时引入过程。Table 29-4. A summary of the event keywords
command      		| alias | explain
-------------------| --- | -----------------%%	|  |	 	得到一个单引号。 All events.%#	|	-serial num		|	事件的序列号。 All events.%a	|	-above win		|	The above field from the event. Configure event.%b	|	-button num		|	按钮号。 Events: ButtonPress and ButtonRelease.%c	|	-count num		|	The count field. Events: Expose and Map.%d	|	-detail value	|	The detail field. Values: NotifyAncestor, NotifyNonlinearVirtual, NotifyDetailNone,NotifyPointer, NotifyInferior, NotifyPointerRoot, NotifyNonlinear, or NotifyVirtual.Events: Enter, Leave, FocusIn, and FocusOut.%f	|	-focus boolean	|	The focus field (0 or 1). Events: Enter and Leave.%h	|	-height num	|	The height field. Events: Configure and Expose.%i	|	| 	The window field from the event, represented as a hexadecimal integer来自事件的窗口域，以十六进制整数表示. All events.%k	|	-keycode num	|	The keycode field. Events: KeyPress and KeyRelease.%m	|	-mode value		|	The mode field. Values: NotifyNormal, NotifyGrab, NotifyUngrab, or NotifyWhileGrabbed. Events: Enter, Leave, FocusIn, and FocusOut.%o	|	-override boolean	|	The override_redirect field. Events: Map, Reparent, and Configure.%p	|	-place value	|	The place field. Values: PlaceOnTop, PlaceOnBottom. Circulate event.%s	|	-state value	|	状态域。事件的十进制字串: ButtonPress, ButtonRelease, Enter, Leave, KeyPress, KeyRelease, and Motion.可视事件的值：Values for the Visibility event: VisibilityUnobscured, VisibilityPartiallyObscured, or VisibilityFullyObscured.%t	|	-time num		|	The time field. All events.%v	|	 |	The value_mask field. Configure event.%w	|	-width num		|	The width field. Events: Configure and Expose.%x	|	-x pixel		|	The X coordinate, widget relative. Mouse events.%y	|	-y pixel		|	The Y coordinate, widget relative. Mouse events.%A	| |	 	The printing character from the event, or {}.Events: KeyPress and KeyRelease.%B	|	-borderwidth num	|	The border width. Configure event.%D	|	-delta value	|	The delta value. MouseWheel event.%E	|	-sendevent bool	|	The send_event field. All events.%K	|	-keysym symbol	|	The keysym from the event. Events: KeyPress and KeyRelease.%N	| |	 	The keysym as a decimal number. Events: KeyPress and KeyRelease.%P	|	| 	The atom name for the property being changed or deleted. Property event.%R	|	-root win		|	The root window ID. All events.%S	|	-subwindow win	|	The subwindow ID. All events.%T	| |	 	The type field. All events.%W	| |	 	The Tk pathname of the widget receiving the event. All events.%X	|	-rootx pixel	|	The x_root field. Relative to the (virtual) root window. Events: ButtonPress, ButtonRelease, KeyPress, KeyRelease, and Motion.%Y	|	-rooty pixel	|	The y_root field. Relative to the (virtual) root window. Events: ButtonPress, ButtonRelease, KeyPress, KeyRelease, and Motion.