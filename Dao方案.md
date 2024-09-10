# 数据库访问类设计方案
Dao接口 
   继承 com.baomidou.mybatisplus.extension.service.IService<实体类> 接口
   方法中的参数用org.apache.ibatis.annotations.Param注解修饰
实体类
   @TableName注解修饰
   主键用@TableId修饰
Dao实现类
   继承com.baomidou.mybatisplus.extension.service.impl.ServiceImpl<自定义mapper, 实体类>
   实现Dao接口
   spring的@Service注解修饰
自定义mapper
   org.apache.ibatis.annotations.Mapper注解修饰
   方法中的参数用org.apache.ibatis.annotations.Param注解修饰
XML形式mapper (路径：resource/mapper/)
   通过IDEA插件MyBatisX快速生成
