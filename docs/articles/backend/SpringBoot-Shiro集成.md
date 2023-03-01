# SpringBoot-Shiro集成

shiro官网集成SpringBoot

https://shiro.apache.org/spring-boot.html



## 1. ShiroConfig编写

```java
@Configuration
public class ShiroConfig {

    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(@Qualifier("securityManager") SecurityManager securityManager) {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        bean.setSecurityManager(securityManager);
        bean.setLoginUrl("/login");
        bean.setSuccessUrl("/index");
        bean.setUnauthorizedUrl("/unauthorizedurl");
        Map<String, String> map = new LinkedHashMap<>();
        map.put("/doLogin", "anon"); //支持匿名访问
        map.put("/**", "authc"); //所有url需要授权才能访问
        bean.setFilterChainDefinitionMap(map);
        return bean;
    }

    @Bean
    public SecurityManager securityManager(@Qualifier("userRealm") Realm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    @Bean
    public Realm userRealm() {
        return new UserRealm();
    }
}
```



## 2. 自定义Realm编写(待完善逻辑)

```java
public class UserRealm extends AuthorizingRealm {
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        //授权(可连库查询，放入缓存）
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //授权 (从缓存取，进行优化)
        String username = (String) token.getPrincipal();
        if (!"root".equals(username)) {
            throw new UnknownAccountException("账户昵称错误");
        }
        return new SimpleAuthenticationInfo(username, "root", "shiro");
    }
}
```



## 3.数据表

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : .
 Source Server Type    : MySQL
 Source Server Version : 50729
 Source Host           : 118.190.91.127:3306
 Source Schema         : admin_sys

 Target Server Type    : MySQL
 Target Server Version : 50729
 File Encoding         : 65001

 Date: 11/03/2020 19:47:25
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for sys_permission
-- ----------------------------
DROP TABLE IF EXISTS `sys_permission`;
CREATE TABLE `sys_permission`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `permission_name` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '权限名称',
  `permission_code` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '权限编码',
  `permission_type` tinyint(2) NOT NULL DEFAULT 0 COMMENT '权限类型\r\n0-菜单\r\n1-按钮',
  `permission_url` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '权限链接',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of sys_permission
-- ----------------------------
INSERT INTO `sys_permission` VALUES (1, '用户查看', 'user:view', 0, 'user/view.html');
INSERT INTO `sys_permission` VALUES (2, '用户编辑', 'user:edit', 1, 'user/edit.do');
INSERT INTO `sys_permission` VALUES (3, '用户删除', 'user:delete', 1, 'user/delete.do');
INSERT INTO `sys_permission` VALUES (4, '用户新增', 'user:add', 1, 'user/add.do');

-- ----------------------------
-- Table structure for sys_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT '角色id',
  `role_name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '角色名',
  `role_code` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '角色编码',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of sys_role
-- ----------------------------
INSERT INTO `sys_role` VALUES (1, '管理员', '10000');
INSERT INTO `sys_role` VALUES (2, '开发', '10001');

-- ----------------------------
-- Table structure for sys_role_permission
-- ----------------------------
DROP TABLE IF EXISTS `sys_role_permission`;
CREATE TABLE `sys_role_permission`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `sys_role_id` bigint(11) NOT NULL COMMENT '角色id',
  `sys_permission_id` bigint(11) NOT NULL COMMENT '权限id',
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `FK_ROLE_PERMISSION_ID`(`sys_role_id`) USING BTREE,
  INDEX `FK_PERMISSION_ROLE_ID`(`sys_permission_id`) USING BTREE,
  CONSTRAINT `FK_ROLE_PERMISSION_ID` FOREIGN KEY (`sys_role_id`) REFERENCES `sys_role` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
  CONSTRAINT `FK_PERMISSION_ROLE_ID` FOREIGN KEY (`sys_permission_id`) REFERENCES `sys_permission` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE = InnoDB AUTO_INCREMENT = 6 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of sys_role_permission
-- ----------------------------
INSERT INTO `sys_role_permission` VALUES (1, 1, 1);
INSERT INTO `sys_role_permission` VALUES (2, 1, 2);
INSERT INTO `sys_role_permission` VALUES (3, 1, 3);
INSERT INTO `sys_role_permission` VALUES (4, 1, 4);
INSERT INTO `sys_role_permission` VALUES (5, 2, 1);

-- ----------------------------
-- Table structure for sys_user
-- ----------------------------
DROP TABLE IF EXISTS `sys_user`;
CREATE TABLE `sys_user`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '昵称',
  `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '密码',
  `salt` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '盐值',
  `state` tinyint(2) NOT NULL DEFAULT 0 COMMENT '状态\r\n0-可用\r\n1-禁用',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of sys_user
-- ----------------------------
INSERT INTO `sys_user` VALUES (1, 'admin', 'admin', 'fdasfjadlskfjlasfj', 0);
INSERT INTO `sys_user` VALUES (2, 'dev', 'dev', 'fdafdasfas', 0);

-- ----------------------------
-- Table structure for sys_user_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_user_role`;
CREATE TABLE `sys_user_role`  (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `sys_user_id` bigint(11) NOT NULL COMMENT '用户id',
  `sys_role_id` bigint(11) NOT NULL COMMENT '角色id',
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `FK_USER_ROLE_ID`(`sys_user_id`) USING BTREE,
  INDEX `FK_ROLE_USER_ID`(`sys_role_id`) USING BTREE,
  CONSTRAINT `FK_USER_ROLE_ID` FOREIGN KEY (`sys_user_id`) REFERENCES `sys_user` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
  CONSTRAINT `FK_ROLE_USER_ID` FOREIGN KEY (`sys_role_id`) REFERENCES `sys_role` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of sys_user_role
-- ----------------------------
INSERT INTO `sys_user_role` VALUES (1, 1, 1);
INSERT INTO `sys_user_role` VALUES (2, 2, 2);

SET FOREIGN_KEY_CHECKS = 1;
```

```sql
-- 查询用户‘admin’所有权限
SELECT
	u.id,
	p.permission_code,
	p.permission_name,
	p.permission_type,
	p.permission_url
FROM
	sys_permission p
	LEFT JOIN sys_role_permission rp ON p.id = rp.sys_permission_id
	LEFT JOIN sys_role r ON r.id = rp.sys_role_id
	LEFT JOIN sys_user_role ur ON ur.sys_role_id = r.id
	LEFT JOIN sys_user u ON ur.sys_user_id = u.id 
WHERE
	u.username = 'dev'
```

