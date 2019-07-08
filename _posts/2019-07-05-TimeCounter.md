---
layout: post
title: 自定义schedule实现定时器
---

## 实现效果如下:  

  ![gif](/images/count.gif)  
## 代码如下:  
{% highlight cpp %}
        LabelTTF *label = LabelTTF::create("23:23:23", "fonts/arial.ttf", 40);
	label->setPosition(Director::getInstance()->getVisibleSize().width/2, 
		Director::getInstance()->getVisibleSize().height / 2);
	addChild(label);
	schedule([label](float dt) {
		int hour = std::atoi(label->getString().substr(0, 2).c_str());
		int minute = std::atoi(label->getString().substr(3, 2).c_str());
		int second = std::atoi(label->getString().substr(6, 2).c_str());
		second -= dt;
		if (second < 0)
		{
			second = minute == 0 && hour == 0 ? 0 : 59;
			minute--;
			if (minute < 0)
			{
				minute = hour == 0 ? 0 : 59;
				hour = --hour < 0 ? 0 : hour;
			}
		}
		label->setString(
      (hour < 10 ? "0" + std::to_string(hour) :
      std::to_string(hour)) + ":" +
      (minute < 10 ? "0" + std::to_string(minute) :
      std::to_string(minute)) + ":" +
      (second < 10 ? "0" + std::to_string(second) :
      std::to_string(second)));
	}, 1.0f,"timeCounter");
{% endhighlight %}
