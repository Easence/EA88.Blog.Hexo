---
title: OpenGL概念篇
categories: 
- OpenGL
tags: 
- OpenGL

---
# 基础概念
### 图元
### 顶点
### 片段
### 管线
### 着色器
- 顶点（vertex）着色器
- 片段（fragment）着色器
- 几何着色器
- GLSL语言
- 属性
- uniform
- 纹理

### 光栅化

# 数学基础
## 法线
### 三角形法线
一个平面的法线是一个长度为1并且垂直于这个平面的向量。 一个三角形的法线是一个长度为1并且垂直于这个三角形的向量。通过简单地将三角形两条边进行叉乘计算，然后归一化：使长度为1。
### 顶点法线
顶点的法线，是包含该顶点的所有三角形的法线的均值。这带来了不少便利–因为在顶点着色器中，我们处理顶点，而不是三角形；所以最好把信息放在顶点上。况且在OpenGL中，我们没有任何办法获得三角形信息。


