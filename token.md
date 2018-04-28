#### token api

#### 1.new token
```sh
type UserAuth struct {
	Token  string `json:"token"`
	Uid    string `json:"uid"`
	Secret string `json:"secret"`
}
POST: localhost:4008/v1/api/token
{
    "uid": "123456",
    "secret": "123",
}

return:
{
    "secret": "4041810df91001619b7b92ed48cf7a7e",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOiIxMjM0NTYiLCJleHAiOjE1MjI4MTI1MzIsImlzcyI6IjEyMzQ1NiJ9.3BRBEyfoie4hUMjhBvnEEn0oIa5YVJrPQTGwrOqCfQs"
}
```

#### 2.check token
```sh
POST: localhost:4008/v1/api/valid
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOiIxMjM0NTYiLCJleHAiOjE1MjI4MTI1MzIsImlzcyI6IjEyMzQ1NiJ9.3BRBEyfoie4hUMjhBvnEEn0oIa5YVJrPQTGwrOqCfQs"
}

本质一样: token异常时，new_uid为空,uid 仍可以获取.
{
    "new_uid": "123456", //newUid, err := utils.AuthToken(ob.Token, secret) 验证成功后，返回的uid
    "uid": "123456"      // 根据token 中payload 解出来uid
}
```


#### jwt 测试代码
```go
func TestCreateTokenMap(t *testing.T) {
	payLoad := map[string]interface{} {
		"uid": "1234",
		"email": "hyhlinux@163.com",
	}
	uid := payLoad["uid"]
	sec := SecSecret(fmt.Sprintf("%v", uid), "testsalt")
	token, err := CreateTokenMap(payLoad, sec, 0)
	if err != nil {
		t.Fatal(err)
	}

	fmt.Printf("CreateTokenMap token:%v\n", token)

	newUid, err := GetUidByMap(token)
	if err != nil {
		t.Fatal(err)
	}
	t.Logf("uid:%v newUid:%v", uid, newUid)
	t.Logf("uid:%v sec:%v", uid, sec)
	data, err := AuthTokenMap(token, sec)
	if err != nil {
		t.Fatal(err)
	}
	t.Logf("data:%v", data)
}
```


#### jwt 生成
```bash
step1.0 用户加密 key, 对part1,part2，进行hash, 生成sign, 作为part3
CreateTokenMap Signed Key:58fded7a087264166c34e6450bb2c941
signed key: []byte(key)
AF SignedString key:[53 56 102 100 101 100 55 97 48 56 55 50 54 52 49 54 54 99 51 52 101 54 52 53 48 98 98 50 99 57 52 49]
i:0 jsonValue:{"alg":"HS256","typ":"JWT"} encode:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
i:1 jsonValue:{"email":"hyhlinux@163.com","exp":1524912431,"uid":"1234"} encode:eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ
token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ
SignedString part1.part2: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ
signingString:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ
Sign hash:CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ
srcHash:[9 105 150 47 205 24 190 198 199 11 177 71 89 186 28 79 67 164 242 157 108 61 233 3 205 132 200 226 166 95 154 4]
SignedString t.Method.Sign(sstr, key) spart3: CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ
token:
    part1: base64url(header)
    part2: base64url(payload)
    part3: hash(part1.part2, key)
CreateTokenMap token:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ.CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ
```

#### jwt 解析
key: 解密key

1.获取part1.part2
2.curSig = hash(part1.part2, key)
3.curSig 是否等于 part3

```bash
BF ParseWithClaims tokenString:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ.CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ claims:map[] keyFunc:0x1346a70Token:&{eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ.CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ 0xc420109f00 map[alg:HS256 typ:JWT] map[uid:1234 email:hyhlinux@163.com exp:1.524912431e+09]  false}
Parts:[eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9 eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ]
err:<nil>
AuthTokenMap Key:58fded7a087264166c34e6450bb2c941
BF Verify:
signingString:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ
key:[53 56 102 100 101 100 55 97 48 56 55 50 54 52 49 54 54 99 51 52 101 54 52 53 48 98 98 50 99 57 52 49]
Signature:CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ
err:<nil>
sig:[9 105 150 47 205 24 190 198 199 11 177 71 89 186 28 79 67 164 242 157 108 61 233 3 205 132 200 226 166 95 154 4]
AF Verify
AF ParseWithClaims tokenString:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ.CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ claims:map[email:hyhlinux@163.com exp:1.524912431e+09 uid:1234] keyFunc:0x1346a70token: &{eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6Imh5aGxpbnV4QDE2My5jb20iLCJleHAiOjE1MjQ5MTI0MzEsInVpZCI6IjEyMzQifQ.CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ 0xc420109f00 map[alg:HS256 typ:JWT] map[exp:1.524912431e+09 uid:1234 email:hyhlinux@163.com] CWmWL80YvsbHC7FHWbocT0Ok8p1sPekDzYTI4qZfmgQ true}
	token_test.go:292: uid:1234 newUid:1234
	token_test.go:293: uid:1234 sec:58fded7a087264166c34e6450bb2c941
	token_test.go:298: data:map[email:hyhlinux@163.com exp:1.524912431e+09 uid:1234]
```