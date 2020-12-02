作业：我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

答：个人觉得，对于多数情况，应该将这个error抛给上层

```
func (d *Dao) FindUserByID(userID string) (*dao.User,error) {
	user,err := DB.Find("user").Where("id = ?", userID).Find(user).Error
	if err != nil {
		if pkgErrors.Cause(err) == data.ErrNoRows {
			return nil, errors.Wrap(err, fmt.Sprintf("not found by user_id: %v", userID))
		}
		return nil, err
	}
	return user,nil
}
```
