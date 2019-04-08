package redlock

import (
	"errors"
	"fmt"
	"github.com/garyburd/redigo/redis"
	uuid "github.com/satori/go.uuid"
)

type Lock struct {
	conn    redis.Conn
	key     string
	token   string
	timeout int
}

var unlockScript = redis.NewScript(1, `
	if redis.call("get", KEYS[1]) == ARGV[1]
	then
		return redis.call("del", KEYS[1])
	else
		return 0
	end
`)

// timeout -- 单位s;
func NewLock(redisConn redis.Conn, key string, timeout int) *Lock {

	locker := &Lock{
		key:     key,
		token:   uuid.NewV4().String(),
		conn:    redisConn,
		timeout: timeout,
	}
	return locker
}

func (this *Lock) Lock() (bool, error) {
	status, err := redis.String(this.conn.Do("SET", this.key, this.token, "EX", int(this.timeout), "NX"))
	if err == redis.ErrNil {
		return false, nil
	}
	if err != nil {
		return false, err
	}
	return status == "OK", nil
}

func (this *Lock) Unlock() (err error) {
	if this == nil {
		return
	}
	value, err := unlockScript.Do(this.conn, this.key, this.token)
	this.close()
	if fmt.Sprint(value) == "0" {
		err = errors.New("超时.")
	}
	return
}

func (this *Lock) close() {
	this.conn.Close()
}

func TryLock(redisConn redis.Conn, key string, timeout int) (*Lock, bool, error) {
	if timeout <= 0 {
		timeout = 60
	}
	lock := NewLock(redisConn, key, timeout)
	ok, err := lock.Lock()
	if !ok || err != nil {
		lock.close()
		return nil, ok, err
	}
	return lock, ok, err
}
