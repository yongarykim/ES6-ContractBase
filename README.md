
### Explanation - ENG
 - BaseContract is base class when making javascript Contract using truffle & ES6-javascript
 - This can be used when using several ES6-javascript like Vue.js, Angular.js and react.js
 - When developing Ethereum dApp, this class helps to map functions between Solidity and Javascript, and
   helps web3 process of MetaMask and etc. 
 - Detail using example can be found on 

### Explanation - KOR
 - truffle을 이용해서 javascript contract를 개발할 때, 상속받아서 쓰도록 만든 기본 class이다.
 - 각종 ES6-javascript language에서 아래 BaseContract를 상속을 받아서 이더리움 dApp개발시 사용할 수 있다.
 - 이더리움 dApp개발시 간단히 solidity의 함수와 js 함수간 매핑하도록 도와주며, metaMask 등의 web3처리를 자동으로 해준다.
 자세한 사용예제는 https://github.com/yongarykim/RSTdAppDevEnv/react-frontend/contracts 를 참고하도록 한다.


### BaseContract code - Using truffle & ES6-javascript

<pre>
/**
 * BaseContract - super class of all contracts (using Web3 & TruffleContract)
 *
 * <History>
 * @author  Yong/Gary Kim (yongary.kim@gmail.com)
 * @date 2018.4.5 - first created
 * @date 2018.6.11 - promisify & getMyAccount added
 * @date 2018.6.13 - contractDeployed added.
 *
 */
import Web3 from 'web3';
import TruffleContract from 'truffle-contract';
import $ from 'jquery';
import { Subject } from 'await-notify';

//make from Web3 to Promise
const promisify = (inner) =>
    new Promise((resolve, reject) =>
        inner((err, res) => {
            if (err) { reject(err) }
            resolve(res);
        })
    );

export default class BaseContract {

    constructor() {
        this.contract = {};
        this.initWeb3();
        //contract.deployed()가 initContract 보다 빨리 호출되는 경우용
        this.eventWaiting = new Subject();
    }

    initWeb3() {
        // Metamask
        console.log("this.web3:")
        console.log(this.web3);
        console.log("window.web3:")
        console.log(window.web3);

        if (typeof window.web3 !== 'undefined') {
            this.web3 = new Web3(window.web3.currentProvider);
        } else {
            //LocalTest  set the provider you want from Web3.providers
            console.log("initWeb3: local");
            this.web3 = new Web3(new Web3.providers.HttpProvider("http://127.0.0.1:7545"));
        }
        console.log(this.web3);
    }

    /* new 이후에 호출 필요 */
    initContract(contractJson) {

        let self = this;
        $.getJSON(contractJson, function (data) {
            // Get the necessary contract artifact file and instantiate it with truffle-contract
            let contractArtifact = data;

            console.log('initContract:' + self.contract);

            self.contract = TruffleContract(contractArtifact);
            self.contract.setProvider(self.web3.currentProvider);

            console.log('initContract out');
            self.eventWaiting.notify(); //contractDeployed가 먼저 불렸을 경우 대비.
        });
    }

    /* contract관련 check함수 */

    /**
     * contractDeployed
     *       - same as contract.deployed,
     *          But waiting initContract to finish
     *       - useful when doing something on screen loading..
     * */
    contractDeployed = async () => {

        console.log('in contractDeployed:');
        if (Object.keys(this.contract).length === 0 ) {//contract Empty = initContract수행중
            console.log(this.contract);

            while (true) {
                await this.eventWaiting.wait();
                console.log('initContract Done');
                break; //exit when notified.
            }
        }
        console.log('out contractDeployed:');
        return this.contract.deployed().then((instance) => {return instance;});

    }


    /* web3/eth Basic함수들 - eth의 sync함수는 값을 못 읽어 오는 경우가 발생가능하므로 async함수를 써야 함. */

    getMyAccount = async () => {
        const accounts = await promisify(cb => this.web3.eth.getAccounts(cb));
        return accounts[0];  //accounts Array의 0번째가 본인 account임.
    }
}
</pre>
