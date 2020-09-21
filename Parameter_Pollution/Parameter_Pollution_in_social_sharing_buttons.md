# Parameter Pollution in social sharing buttons

Hi Guys,
Though it is not severe bug.But still some organizations take this seriously.

## Steps :

```
1.Browse through your target.
  say https://target.com
2.Find a article or blog present on target website which must have a link to share that blog on different social networks such as
  Facebook,Twitter etc.
3.Let's say we got and article with url:
  https://taget.com/how-to-hunt 
  then just appened it with payload ?&u=https://attacker.com/vaya&text=another_site:https://attacker.com/vaya
  so our url will become 
  https://taget.com/how-to-hunt?&u=https://attacker.com/vaya&text=another_site:https://attacker.com/vaya
4.Now hit enter with the abover url and just click on share with social media.
  Just observe the content if it is including our payload i.e. https://attacker.com
  Then it is vulnerable or else try next target.
```  
## References:
* https://hackerone.com/reports/105953
* Google
  
## Author
* [KenAdams000](https://twitter.com/KenAdams000)
