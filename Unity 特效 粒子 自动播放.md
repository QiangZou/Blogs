# Unity 特效 粒子 自动播放

## 问题由来
- 在unity不运行状态 需要展示多个特效
- 观察只有选中粒子对象才会播放

## 解决方法
- 自动获取场景所有粒子对象
- 赋值给Selection.objects


```
ParticleSystem[] particleSystemList = 根目录.transform.GetComponentsInChildren<ParticleSystem>(true);
Object[] objList = new Object[particleSystemList.Length];
for (int i = 0; i < particleSystemList.Length; i++)
{
    objList[i] = particleSystemList[i].gameObject;
}

Selection.objects = objList;
```