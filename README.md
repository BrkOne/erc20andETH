# Simple Ethereum ERC20 token php library

This library provides simple way to interact with Ethereum ERC20 token.  
By default, supports all ERC20 Standard functions (like balanceOf, transfer, transferFrom, approve, allowance, decimal, name, ...) also can be extends to support other contracts as well.

## Installation
`composer require harensarma/erc20-php`

## Usage
There are two ways to use:
#### 1- Make a new class for your token and specified their functions
#### 2- Use general class with all standard functions

See below to find out more


### 1-Make a new class for your token
Simply create a new class inherits from `\Placecodex\Ethereum\Foundation\StandardERC20Token`

in below sample we create a new class for Tether (USDT)
```
class USDT extends \Placecodex\Ethereum\Foundation\StandardERC20Token 
{
    protected $contractAddress = "0xdac17f958d2ee523a2206206994597c13d831ec7";  
}
```
Then for use create new instantiate from your class and

```
$tether = new USDT("https://mainnet.infura.io/v3/API_KEY");
var_dump($tether->name());
var_dump($tether->decimals());
```

### 2- Use general class

```
$token = new \Placecodex\Ethereum\Token("0xdac17f958d2ee523a2206206994597c13d831ec7", "https://mainnet.infura.io/v3/API_KEY");
var_dump($token->name());
```


### Connection Timeout

Connection timeout can be set by last parameter of token class

```
$timeout  = 3; //secs
$tether = new USDT("https://mainnet.infura.io/v3/API_KEY",$timeout);
```
OR
```
$timeout  = 3; //secs
$tether = new \Placecodex\Ethereum\Token("0xdac17f958d2ee523a2206206994597c13d831ec7", "https://mainnet.infura.io/v3/API_KEY", $timeout);
```

## Ethereum RPC Client
For connect to Ethereum blockchain you need an Ethereum node; [Infura](https://infura.io/) is a simple and fast solution, however you can launch you [Geth](https://geth.ethereum.org/) node


## ERC20 Token `transferFrom`
ERC20 transaction fee needs to be paid in `ETH`. In some situation your app needs to pay this fee behalf of user.  
Suppose, user A have a key pair (private, public) and all their transaction is limited to usdt. User A needs to send 10 usdt, but he/she haven't ETH to pay transaction fee.  
In these cases your app should pay fee behalf of users.      
`transferFrom` is a good solution in these cases.


## `transferFrom` Flow:
1.First, Using `approve` method to grant permission to a delegator.  
2.Then, Using `transferFrom` method to make transaction behalf of user. 

*In Action*
```
$owner_private = '0xcf29c83a88e23d0b9e676beca426490bf79aca71e9d24f79a99d30c48292e1e3';
$owner_address = '0xA7e5F270c27E9d33911EE7D50D8E814f793d2760';

$myapp_private = '0xa6b6be193bfeac6160178ee6e1435609ae566a9054715e0802e4c3b39bb94e83';
$myapp_address = '0x8dC9b3c20795815aa063FEdBE8E564566CEc1893';

$to_address = '0x245013F05DdA116142Ca8db205ec4F8C780E3DcB';

//by this method we allow $myapp_address to send upto 99999 token behalf of $owner_address
$approve_tx    = $token->approve($owner_address, $myapp_address, 99999);
$approve_tx_id = $approve_tx->sign($owner_private)
                            ->send()
;


//the magic is here, $myapp_address send 10 tokens behalf of user and $myapp_address pay transaction fee
$transfer_tx    = $token->transferFrom($myapp_address, $owner_address, $to_address, 10);
$transfer_tx_id = $transfer_tx->sign($myapp_private)
                              ->send()
;

```

### `allowance` to check how much transferFrom remain

`$remain = $token->allowance($owner_address, $myapp_address);`

*Notices:*  
`approve` method not need to be used on every transaction.  
To revoke `transferFrom` permission call `$token->approve($owner_address, $myapp_address, 0)` by amount `0`.  
# erc20andETH

# Incase of BNB or BEP-20 Transaction

Go to the file: web3p/ethereum-tx/src/Transaction.php  and put the below:

<?php

/**
 * This file is part of ethereum-tx package.
 *
 * (c) Kuan-Cheng,Lai <alk03073135@gmail.com>
 *
 * @author Peter Lai <alk03073135@gmail.com>
 * @license MIT
 */

namespace Web3p\EthereumTx;

use InvalidArgumentException;
use RuntimeException;
use Web3p\RLP\RLP;
use Elliptic\EC;
use Elliptic\EC\KeyPair;
use ArrayAccess;
use Web3p\EthereumUtil\Util;

/**
 * It's a instance for generating/serializing ethereum transaction.
 *
 * ```php
 * use Web3p\EthereumTx\Transaction;
 *
 * // generate transaction instance with transaction parameters
 * $transaction = new Transaction([
 *     'nonce' => '0x01',
 *     'from' => '0xb60e8dd61c5d32be8058bb8eb970870f07233155',
 *     'to' => '0xd46e8dd67c5d32be8058bb8eb970870f07244567',
 *     'gas' => '0x76c0',
 *     'gasPrice' => '0x9184e72a000',
 *     'value' => '0x9184e72a',
 *     'chainId' => 1, // optional
 *     'data' => '0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675'
 * ]);
 *
 * // generate transaction instance with hex encoded transaction
 * $transaction = new Transaction('0xf86c098504a817c800825208943535353535353535353535353535353535353535880de0b6b3a76400008025a028ef61340bd939bc2195fe537567866003e1a15d3c71ff63e1590620aa636276a067cbe9d8997f761aecb703304b3800ccf555c9f3dc64214b297fb1966a3b6d83');
 * ```
 *
 * ```php
 * After generate transaction instance, you can sign transaction with your private key.
 * <code>
 * $signedTransaction = $transaction->sign('your private key');
 * ```
 *
 * Then you can send serialized transaction to ethereum through http rpc with web3.php.
 * ```php
 * $hashedTx = $transaction->serialize();
 * ```
 *
 * @author Peter Lai <alk03073135@gmail.com>
 * @link https://www.web3p.xyz
 * @filesource https://github.com/web3p/ethereum-tx
 */
class Transaction implements ArrayAccess
{
    /**
     * Attribute map for keeping order of transaction key/value
     *
     * @var array
     */
    protected $attributeMap = [
        'from' => [
            'key' => -1
        ],
        'chainId' => [
            'key' => -2
        ],
        'nonce' => [
            'key' => 0,
            'length' => 32,
            'allowLess' => true,
            'allowZero' => false
        ],
        'gasPrice' => [
            'key' => 1,
            'length' => 32,
            'allowLess' => true,
            'allowZero' => false
        ],
        'gasLimit' => [
            'key' => 2,
            'length' => 32,
            'allowLess' => true,
            'allowZero' => false
        ],
        'gas' => [
            'key' => 2,
            'length' => 32,
            'allowLess' => true,
            'allowZero' => false
        ],
        'to' => [
            'key' => 3,
            'length' => 20,
            'allowZero' => true,
        ],
        'value' => [
            'key' => 4,
            'length' => 32,
            'allowLess' => true,
            'allowZero' => false
        ],
        'data' => [
            'key' => 5,
            'allowLess' => true,
            'allowZero' => true
        ],
        'v' => [
            'key' => 6,
            'allowZero' => true
        ],
        'r' => [
            'key' => 7,
            'length' => 32,
            'allowZero' => true
        ],
        's' => [
            'key' => 8,
            'length' => 32,
            'allowZero' => true
        ]
    ];

    /**
     * Raw transaction data
     *
     * @var array
     */
    protected $txData = [];

    /**
     * RLP encoding instance
     *
     * @var \Web3p\RLP\RLP
     */
    protected $rlp;

    /**
     * secp256k1 elliptic curve instance
     *
     * @var \Elliptic\EC
     */
    protected $secp256k1;

    /**
     * Private key instance
     *
     * @var \Elliptic\EC\KeyPair
     */
    protected $privateKey;

    /**
     * Ethereum util instance
     *
     * @var \Web3p\EthereumUtil\Util
     */
    protected $util;

    /**
     * construct
     *
     * @param array|string $txData
     * @return void
     */
    public function __construct($txData=[])
    {
        $this->rlp = new RLP;
        $this->secp256k1 = new EC('secp256k1');
        $this->util = new Util;

        if (is_array($txData)) {
            foreach ($txData as $key => $data) {
                $this->offsetSet($key, $data);
            }
        } elseif (is_string($txData)) {
            $tx = [];

            if ($this->util->isHex($txData)) {
                $txData = $this->rlp->decode($txData);

                foreach ($txData as $txKey => $data) {
                    if (is_int($txKey)) {
                        $hexData = $data;

                        if (strlen($hexData) > 0) {
                            $tx[$txKey] = '0x' . $hexData;
                        } else {
                            $tx[$txKey] = $hexData;
                        }
                    }
                }
            }
            $this->txData = $tx;
        }
    }

    /**
     * Return the value in the transaction with given key or return the protected property value if get(property_name} function is existed.
     *
     * @param string $name key or protected property name
     * @return mixed
     */
    public function __get(string $name)
    {
        $method = 'get' . ucfirst($name);

        if (method_exists($this, $method)) {
            return call_user_func_array([$this, $method], []);
        }
        return $this->offsetGet($name);
    }

    /**
     * Set the value in the transaction with given key or return the protected value if set(property_name} function is existed.
     *
     * @param string $name key, eg: to
     * @param mixed value
     * @return void
     */
    public function __set(string $name, $value)
    {
        $method = 'set' . ucfirst($name);

        if (method_exists($this, $method)) {
            return call_user_func_array([$this, $method], [$value]);
        }
        return $this->offsetSet($name, $value);
    }

    /**
     * Return hash of the ethereum transaction without signature.
     *
     * @return string hex encoded of the transaction
     */
    public function __toString()
    {
        return $this->hash(false);
    }

    /**
     * Set the value in the transaction with given key.
     *
     * @param string $offset key, eg: to
     * @param string value
     * @return void
     */
    public function offsetSet($offset, $value)
    {
        $txKey = isset($this->attributeMap[$offset]) ? $this->attributeMap[$offset] : null;

        if (is_array($txKey)) {
            $checkedValue = ($value) ? (string) $value : '';
            $isHex = $this->util->isHex($checkedValue);
            $checkedValue = $this->util->stripZero($checkedValue);

            if (!isset($txKey['allowLess']) || (isset($txKey['allowLess']) && $txKey['allowLess'] === false)) {
                // check length
                if (isset($txKey['length'])) {
                    if ($isHex) {
                        if (strlen($checkedValue) > $txKey['length'] * 2) {
                            throw new InvalidArgumentException($offset . ' exceeds the length limit.');
                        }
                    } else {
                        if (strlen($checkedValue) > $txKey['length']) {
                            throw new InvalidArgumentException($offset . ' exceeds the length limit.');
                        }
                    }
                }
            }
            if (!isset($txKey['allowZero']) || (isset($txKey['allowZero']) && $txKey['allowZero'] === false)) {
                // check zero
                if (preg_match('/^0*$/', $checkedValue) === 1) {
                    // set value to empty string
                    $value = '';
                }
            }
            $this->txData[$txKey['key']] = $value;
        }
    }

    /**
     * Return whether the value is in the transaction with given key.
     *
     * @param string $offset key, eg: to
     * @return bool
     */
    public function offsetExists($offset)
    {
        $txKey = isset($this->attributeMap[$offset]) ? $this->attributeMap[$offset] : null;

        if (is_array($txKey)) {
            return isset($this->txData[$txKey['key']]);
        }
        return false;
    }

    /**
     * Unset the value in the transaction with given key.
     *
     * @param string $offset key, eg: to
     * @return void
     */
    public function offsetUnset($offset)
    {
        $txKey = isset($this->attributeMap[$offset]) ? $this->attributeMap[$offset] : null;

        if (is_array($txKey) && isset($this->txData[$txKey['key']])) {
            unset($this->txData[$txKey['key']]);
        }
    }

    /**
     * Return the value in the transaction with given key.
     *
     * @param string $offset key, eg: to
     * @return mixed value of the transaction
     */
    public function offsetGet($offset)
    {
        if ($offset == 'chainId') {
            return 56; // In case of BNB or BEP-20 transaction.
        }

        $txKey = isset($this->attributeMap[$offset]) ? $this->attributeMap[$offset] : null;

        if (is_array($txKey) && isset($this->txData[$txKey['key']])) {
            return $this->txData[$txKey['key']];
        }
        return null;
    }

    /**
     * Return raw ethereum transaction data.
     *
     * @return array raw ethereum transaction data
     */
    public function getTxData()
    {
        return $this->txData;
    }

    /**
     * RLP serialize the ethereum transaction.
     *
     * @return \Web3p\RLP\RLP\Buffer serialized ethereum transaction
     */
    public function serialize()
    {
        $chainId = $this->offsetGet('chainId');

        // sort tx data
        if (ksort($this->txData) !== true) {
            throw new RuntimeException('Cannot sort tx data by keys.');
        }
        if ($chainId && $chainId > 0) {
            $txData = array_fill(0, 9, '');
        } else {
            $txData = array_fill(0, 6, '');
        }
        foreach ($this->txData as $key => $data) {
            if ($key >= 0) {
                $txData[$key] = $data;
            }
        }
        return $this->rlp->encode($txData);
    }

    /**
     * Sign the transaction with given hex encoded private key.
     *
     * @param string $privateKey hex encoded private key
     * @return string hex encoded signed ethereum transaction
     */
    public function sign(string $privateKey)
    {
        if ($this->util->isHex($privateKey)) {
            $privateKey = $this->util->stripZero($privateKey);
            $ecPrivateKey = $this->secp256k1->keyFromPrivate($privateKey, 'hex');
        } else {
            throw new InvalidArgumentException('Private key should be hex encoded string');
        }
        $txHash = $this->hash(false);
        $signature = $ecPrivateKey->sign($txHash, [
            'canonical' => true
        ]);
        $r = $signature->r;
        $s = $signature->s;
        $v = $signature->recoveryParam + 35;

        $chainId = $this->offsetGet('chainId');

        if ($chainId && $chainId > 0) {
            $v += (int) $chainId * 2;
        }

        $this->offsetSet('r', '0x' . $r->toString(16));
        $this->offsetSet('s', '0x' . $s->toString(16));
        $this->offsetSet('v', $v);
        $this->privateKey = $ecPrivateKey;

        return $this->serialize();
    }

    /**
     * Return hash of the ethereum transaction with/without signature.
     *
     * @param bool $includeSignature hash with signature
     * @return string hex encoded hash of the ethereum transaction
     */
    public function hash(bool $includeSignature=false)
    {
        $chainId = $this->offsetGet('chainId');

        // sort tx data
        if (ksort($this->txData) !== true) {
            throw new RuntimeException('Cannot sort tx data by keys.');
        }
        if ($includeSignature) {
            $txData = $this->txData;
        } else {
            $rawTxData = $this->txData;

            if ($chainId && $chainId > 0) {
                $v = (int) $chainId;
                $this->offsetSet('r', '');
                $this->offsetSet('s', '');
                $this->offsetSet('v', $v);
                $txData = array_fill(0, 9, '');
            } else {
                $txData = array_fill(0, 6, '');
            }

            foreach ($this->txData as $key => $data) {
                if ($key >= 0) {
                    $txData[$key] = $data;
                }
            }
            $this->txData = $rawTxData;
        }
        $serializedTx = $this->rlp->encode($txData);

        return $this->util->sha3(hex2bin($serializedTx));
    }

    /**
     * Recover from address with given signature (r, s, v) if didn't set from.
     *
     * @return string hex encoded ethereum address
     */
    public function getFromAddress()
    {
        $from = $this->offsetGet('from');

        if ($from) {
            return $from;
        }
        if (!isset($this->privateKey) || !($this->privateKey instanceof KeyPair)) {
            // recover from hash
            $r = $this->offsetGet('r');
            $s = $this->offsetGet('s');
            $v = $this->offsetGet('v');
            $chainId = $this->offsetGet('chainId');

            if (!$r || !$s) {
                throw new RuntimeException('Invalid signature r and s.');
            }
            $txHash = $this->hash(false);

            if ($chainId && $chainId > 0) {
                $v -= ($chainId * 2);
            }
            $v -= 35;
            $publicKey = $this->secp256k1->recoverPubKey($txHash, [
                'r' => $r,
                's' => $s
            ], $v);
            $publicKey = $publicKey->encode('hex');
        } else {
            $publicKey = $this->privateKey->getPublic(false, 'hex');
        }
        $from = '0x' . substr($this->util->sha3(substr(hex2bin($publicKey), 1)), 24);

        $this->offsetSet('from', $from);
        return $from;
    }
}
