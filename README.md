[![Build Status](https://drone.io/github.com/pdf/golifx/status.png)](https://drone.io/github.com/pdf/golifx/latest) [![GoDoc](https://godoc.org/github.com/pdf/golifx?status.svg)](http://godoc.org/github.com/pdf/golifx) ![License-MIT](http://img.shields.io/badge/license-MIT-red.svg)

__Note:__ This library is at a moderately early stage - everything should work,
but the V2 protocol implementation needs documentation and testing.

If you have Go installed, you can install the `lifx` CLI application like so:

```shell
go get -u github.com/pdf/golifx/cmd/lifx
```

The `lifx` command will be available at `${GOPATH}/bin/lifx`

# golifx
--
    import "github.com/pdf/golifx"

Package golifx provides a simple Go interface to the LIFX LAN protocol.

Based on the protocol documentation available at: http://lan.developer.lifx.com/

Also included in cmd/lifx is a small CLI utility that allows interacting with
your LIFX devices on the LAN.

In various parts of this package you may find references to a Device or a Light.
The LIFX protocol makes room for future non-light devices by making a light a
superset of a device, so a Light is a Device, but a Device is not necessarily a
Light. At this stage, LIFX only produces lights though, so they are the only
type of device you will interact with.

## Usage

```go
const (
	// VERSION of this library
	VERSION = `0.1.0`
)
```

#### func  SetLogger

```go
func SetLogger(logger common.Logger)
```
SetLogger allows assigning a custom levelled logger that conforms to the
common.Logger interface. To capture logs generated during client creation, this
should be called before creating a Client. Defaults to common.StubLogger, which
does no logging at all.

#### type Client

```go
type Client struct {
	sync.RWMutex
}
```

Client provides a simple interface for interacting with LIFX devices. Client can
not be instantiated manually or it will not function - always use NewClient() to
obtain a Client instance.

#### func  NewClient

```go
func NewClient(p common.Protocol) (*Client, error)
```
NewClient returns a pointer to a new Client and any error that occurred
initializing the client, using the protocol p. It also kicks off a discovery
run.

#### func (*Client) Close

```go
func (c *Client) Close() error
```
Close signals the termination of this client, and cleans up resources

#### func (*Client) CloseSubscription

```go
func (c *Client) CloseSubscription(sub *common.Subscription) error
```
CloseSubscription is a callback for handling the closing of subscriptions.

#### func (*Client) GetDeviceByID

```go
func (c *Client) GetDeviceByID(id uint64) (common.Device, error)
```
GetDeviceByID looks up a device by it's id and returns a common.Device. May
return a common.ErrNotFound error if the lookup times out without finding the
device.

#### func (*Client) GetDeviceByLabel

```go
func (c *Client) GetDeviceByLabel(label string) (common.Device, error)
```
GetDeviceByLabel looks up a device by it's label and returns a common.Device.
May return a common.ErrNotFound error if the lookup times out without finding
the device.

#### func (*Client) GetDevices

```go
func (c *Client) GetDevices() ([]common.Device, error)
```
GetDevices returns a slice of all devices known to the client, or
common.ErrNotFound if no devices are currently known.

#### func (*Client) GetLightByID

```go
func (c *Client) GetLightByID(id uint64) (light common.Light, err error)
```
GetLightByID looks up a light by it's id and returns a common.Light. May return
a common.ErrNotFound error if the lookup times out without finding the light, or
common.ErrDeviceInvalidType if the device exists but is not a light.

#### func (*Client) GetLightByLabel

```go
func (c *Client) GetLightByLabel(label string) (common.Light, error)
```
GetLightByLabel looks up a light by it's label and returns a common.Light. May
return a common.ErrNotFound error if the lookup times out without finding the
light, or common.ErrDeviceInvalidType if the device exists but is not a light.

#### func (*Client) GetLights

```go
func (c *Client) GetLights() (lights []common.Light, err error)
```
GetLights returns a slice of all lights known to the client, or
common.ErrNotFound if no lights are currently known.

#### func (*Client) GetRetryInterval

```go
func (c *Client) GetRetryInterval() *time.Duration
```
GetRetryInterval returns the currently configured retry interval for operations
on this client

#### func (*Client) GetTimeout

```go
func (c *Client) GetTimeout() *time.Duration
```
GetTimeout returns the currently configured timeout period for operations on
this client

#### func (*Client) NewSubscription

```go
func (c *Client) NewSubscription() (*common.Subscription, error)
```
NewSubscription returns a new *common.Subscription for receiving events from
this client.

#### func (*Client) SetColor

```go
func (c *Client) SetColor(color common.Color, duration time.Duration) error
```
SetColor broadcasts a request to change the color of all devices on the network.

#### func (*Client) SetDiscoveryInterval

```go
func (c *Client) SetDiscoveryInterval(interval time.Duration) error
```
SetDiscoveryInterval causes the client to discover devices and state every
interval. You should set this to a non-zero value for any long-running process,
otherwise devices will only be discovered once.

#### func (*Client) SetPower

```go
func (c *Client) SetPower(state bool) error
```
SetPower broadcasts a request to change the power state of all devices on the
network. A state of true requests power on, and a state of false requests power
off.

#### func (*Client) SetPowerDuration

```go
func (c *Client) SetPowerDuration(state bool, duration time.Duration) error
```
SetPowerDuration broadcasts a request to change the power state of all devices
on the network, transitioning over the specified duration. A state of true
requests power on, and a state of false requests power off. Not all device types
support transitioning, so if you wish to change the state of all device types,
you should use SetPower instead.

#### func (*Client) SetRetryInterval

```go
func (c *Client) SetRetryInterval(retryInterval time.Duration)
```
SetRetryInterval sets the retry interval for operations on this client. If a
timeout has been set, and the retry interval exceeds the timeout, the retry
interval will be set to half the timeout

#### func (*Client) SetTimeout

```go
func (c *Client) SetTimeout(timeout time.Duration)
```
SetTimeout sets the time that client operations wait for results before
returning an error. The special value of 0 may be set to disable timeouts, and
all operations will wait indefinitely, but this is not recommended.
# common
--
    import "github.com/pdf/golifx/common"

Package common contains common elements for the golifx client and protocols

## Usage

```go
const (
	// DefaultTimeout is the default duration after which operations time out
	DefaultTimeout = 2 * time.Second
	// DefaultRetryInterval is the default interval at which operations are
	// retried
	DefaultRetryInterval = 500 * time.Millisecond
)
```

```go
var (
	// ErrNotFound not found
	ErrNotFound = errors.New(`Not found`)
	// ErrProtocol protocol error
	ErrProtocol = errors.New(`Protocol error`)
	// ErrDuplicate already exists
	ErrDuplicate = errors.New(`Already exists`)
	// ErrInvalidArgument invalid argument
	ErrInvalidArgument = errors.New(`Invalid argument`)
	// ErrClosed connection closed
	ErrClosed = errors.New(`Connection closed`)
	// ErrTimeout timed out
	ErrTimeout = errors.New(`Timed out`)
	// ErrDeviceInvalidType invalid device type
	ErrDeviceInvalidType = errors.New(`Invalid device type`)
)
```

#### func  SetLogger

```go
func SetLogger(logger Logger)
```
SetLogger wraps the supplied logger with a logPrefixer to denote golifx logs

#### type Client

```go
type Client interface {
	GetTimeout() *time.Duration
	GetRetryInterval() *time.Duration
}
```

Client defines the interface required by protocols

#### type Color

```go
type Color struct {
	Hue        uint16 // range 0 to 65535
	Saturation uint16 // range 0 to 65535
	Brightness uint16 // range 0 to 65535
	Kelvin     uint16 // range 2500° (warm) to 9000° (cool)
}
```

Color is used to represent the color and color temperature of a light. The color
is represented as an HSB (Hue, Saturation, Brightness) value. The color
temperature is represented in K (Kelvin) and is used to adjust the warmness /
coolness of a white light, which is most obvious when saturation is close zero.

#### type Device

```go
type Device interface {
	SubscriptionTarget
	// Returns the ID for the device
	ID() uint64

	// Returns the label for the device
	GetLabel() (string, error)
	// Sets the label for the device
	SetLabel(label string) error
	// Returns the power state of the device, true for on, false for off
	GetPower() (bool, error)
	// Sets the power state of the device, true for on, false for off
	SetPower(state bool) error
}
```

Device represents a generic LIFX device

#### type ErrNotImplemented

```go
type ErrNotImplemented struct {
	Method string
}
```

ErrNotImplemented not implemented

#### func (*ErrNotImplemented) Error

```go
func (e *ErrNotImplemented) Error() string
```
Error satisfies the error interface

#### type EventExpiredDevice

```go
type EventExpiredDevice struct {
	Device Device
}
```

EventExpiredDevice is emitted by a Client when a Device is no longer known

#### type EventNewDevice

```go
type EventNewDevice struct {
	Device Device
}
```

EventNewDevice is emitted by a Client when it discovers a new Device

#### type EventUpdateColor

```go
type EventUpdateColor struct {
	Color Color
}
```

EventUpdateColor is emitted by a Light when it's power state is updated

#### type EventUpdateLabel

```go
type EventUpdateLabel struct {
	Label string
}
```

EventUpdateLabel is emitted by a Device when it's label is updated

#### type EventUpdatePower

```go
type EventUpdatePower struct {
	Power bool
}
```

EventUpdatePower is emitted by a Device when it's power state is updated

#### type Light

```go
type Light interface {
	// A light is a superset of the Device interface
	Device
	// SetColor changes the color of the light, transitioning over the specified
	// duration
	SetColor(color Color, duration time.Duration) error
	// GetColor returns the current color of the light
	GetColor() (Color, error)
	// SetPowerDuration sets the power of the light, transitioning over the
	// speficied duration, state is true for on, false for off.
	SetPowerDuration(state bool, duration time.Duration) error
}
```

Light represents a LIFX light device

#### type Logger

```go
type Logger interface {
	// Debugf handles debug level messages
	Debugf(format string, args ...interface{})
	// Infof handles info level messages
	Infof(format string, args ...interface{})
	// Warnf handles warn level messages
	Warnf(format string, args ...interface{})
	// Errorf handles error level messages
	Errorf(format string, args ...interface{})
	// Fatalf handles fatal level messages, and must exit the application
	Fatalf(format string, args ...interface{})
	// Panicf handles debug level messages, and must panic the application
	Panicf(format string, args ...interface{})
}
```

Logger represents a minimal levelled logger

```go
var (
	// Log holds the global logger used by golifx, can be set via SetLogger() in
	// the golifx package
	Log Logger
)
```

#### type Protocol

```go
type Protocol interface {
	SubscriptionTarget
	// SetClient sets the client on the protocol for bi-directional
	// communication
	SetClient(client Client)
	// Discover initiates device discovery, this may be a noop in some future
	// protocol versions.  This is called immediately when the client connects
	// to the protocol
	Discover() error
	// Close closes the protocol driver, no further communication with the
	// protocol is possible
	Close() error

	// SetPower sets the power state globally, on all devices
	SetPower(state bool) error
	// SetPowerDuration sets the power state globally, on all lights, over the
	// specified duration
	SetPowerDuration(state bool, duration time.Duration) error
	// SetColor changes the color globally, on all lights, over the specified
	// duration
	SetColor(color Color, duration time.Duration) error
}
```

Protocol defines the interface between the Client and a protocol implementation

#### type StubLogger

```go
type StubLogger struct{}
```

StubLogger satisfies the Logger interface, and simply does nothing with received
messages

#### func (*StubLogger) Debugf

```go
func (l *StubLogger) Debugf(format string, args ...interface{})
```
Debugf handles debug level messages

#### func (*StubLogger) Errorf

```go
func (l *StubLogger) Errorf(format string, args ...interface{})
```
Errorf handles error level messages

#### func (*StubLogger) Fatalf

```go
func (l *StubLogger) Fatalf(format string, args ...interface{})
```
Fatalf handles fatal level messages, exits the application

#### func (*StubLogger) Infof

```go
func (l *StubLogger) Infof(format string, args ...interface{})
```
Infof handles info level messages

#### func (*StubLogger) Panicf

```go
func (l *StubLogger) Panicf(format string, args ...interface{})
```
Panicf handles debug level messages, and panics the application

#### func (*StubLogger) Warnf

```go
func (l *StubLogger) Warnf(format string, args ...interface{})
```
Warnf handles warn level messages

#### type Subscription

```go
type Subscription struct {
}
```

Subscription exposes an event channel for consumers, and attaches to a
SubscriptionTarget, that will feed it with events

#### func  NewSubscription

```go
func NewSubscription(target SubscriptionTarget) *Subscription
```
NewSubscription returns a *Subscription attached to the specified target

#### func (*Subscription) Close

```go
func (s *Subscription) Close()
```
Close cleans up resources and notifies the target that the subscription should
no longer be used. It is important to close subscriptions when you are done with
them to avoid blocking operations.

#### func (*Subscription) Events

```go
func (s *Subscription) Events() <-chan interface{}
```
Events returns a chan reader for reading events published to this subscription

#### func (*Subscription) ID

```go
func (s *Subscription) ID() string
```
ID returns the unique ID for this subscription

#### func (*Subscription) Write

```go
func (s *Subscription) Write(event interface{}) error
```
Write pushes an event onto the events channel

#### type SubscriptionTarget

```go
type SubscriptionTarget interface {
	NewSubscription() (*Subscription, error)
	CloseSubscription(*Subscription) error
}
```

SubscriptionTarget defines the interface between a subscription and it's target
object
