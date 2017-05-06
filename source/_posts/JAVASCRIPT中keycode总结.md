---
title: JAVASCRIPT中keycode总结
date: 2011-02-05 23:39
categories: 
- Javascript
tags: 
- JavaScript
---
Keypress 的KeyCode: 
<table class="bbcode">
 <tbody>
  <tr>
   <td>-</td>
   <td>小键盘</td>
   <td>大键盘</td>
  </tr> 
  <tr>
   <td>“-”</td>
   <td>45</td>
   <td>45</td>
  </tr> 
  <tr>
   <td>“.”</td>
   <td> 46</td>
   <td>46</td>
  </tr> 
  <tr>
   <td>0~9</td>
   <td>48~57</td>
   <td>48~57</td>
  </tr>&nbsp; 
  <tr>
   <td>a~z</td>
   <td>-</td>
   <td> 97~122</td>
  </tr> 
  <tr>
   <td>“`”</td>
   <td>-</td>
   <td>96</td>
  </tr>
 </tbody>
</table> 

Keydown 的keycode: 

<table class="bbcode"> 
 <tbody>
  <tr>
   <td>-</td>
   <td>小键盘</td>
   <td>大键盘</td>
  </tr>&nbsp; 
  <tr>
   <td>“-”</td>
   <td>109</td>
   <td> 189</td>
  </tr> 
  <tr>
   <td>“.”</td>
   <td>110</td>
   <td>190</td>
  </tr> 
  <tr>
   <td>0~9</td>
   <td>96~105</td>
   <td>48~57</td>
  </tr> 
  <tr>
   <td>左,上,右,下</td>
   <td>-</td>
   <td>37~40</td>
  </tr>
 </tbody>
</table> 
Keyup的keycode同keydown相同,注意keydown总是在keypress之前触发,用keyup可以获得用户按键后输入。 
