# upwork-client
Test



import React from 'react';
import {Row,Col,Table,Form,Button,Alert,Card,Container,Nav} from 'react-bootstrap';
import Candle from '../Candle';
import {Link} from 'react-router-dom';
import { throws } from 'assert';
import { ToastContainer, toast } from 'react-toastify';
import ScaleLoader from "react-spinners/ScaleLoader";
import {FaWallet,FaRegTrashAlt} from 'react-icons/fa';
import Skeleton from 'react-loading-skeleton';
import io from "socket.io-client";

const styles = {
    textAlign: 'center',
    Color: '#1562f0 !important'
  };
var socket;
var connectionOptions = {
    "force new connection" :true,
    "reconnectionAttempts" : "Infinity",
    "timeout" : 10000,
    "transports" : ["websocket"]
};

const axios = require('axios');

export default class Trade extends React.Component{

    constructor(props){
        super(props);
        this.state={
            endpoint: 'https://greyz-socket.herokuapp.com/',
            Sellloading:true,
            Buyloading:true,
            Executedloading:true,
            asset:this.props.asset,
            userId:this.props.userId,
            buyOrders:[],
            sellOrders:[],
            executedOrders:[],
            Buyprice:0,
            Buyamount:0,
            Sellprice:0,
            Sellamount:0,
            greyzBal:0,
            assetBal:0,
            sellButton:false,
            buyButton:false,
            buyError:false,
            buyErrorMsg:'',
            sellError:false,
            sellErrorMsg:'',
            isHigh:true,
            isEmpty:false,
            high:0,
            change:0,
            Low:0,
            Volume:0,
            isMySell:false,
            isMyBuy:false,
            isMySellLoading:false,
            isMyBuyLoading:false,
            mySellOrders:[],
            myBuyOrders:[],
            mySellNav:'orderBook',
            myBuyNav:'orderBook',
        }

        this.fetchBuyOrders = this.fetchBuyOrders.bind(this);
        this.fetchSellOrders = this.fetchSellOrders.bind(this);
        this.handlePairs=this.handlePairs.bind(this);
        this.submitBuyOrder=this.submitBuyOrder.bind(this);
        this.submitSellOrder=this.submitSellOrder.bind(this);
        this.handleChange=this.handleChange.bind(this);
        this.sendSocketIO = this.sendSocketIO.bind(this);
        this.getBuyOrders = this.getBuyOrders.bind(this);
        this.getSellOrders = this.getSellOrders.bind(this);
        this.fetchBalance=this.fetchBalance.bind(this);
        this.fetchPrice=this.fetchPrice.bind(this);
        this.fetchMySell = this.fetchMySell.bind(this);
        this.fetchMyBuy = this.fetchMyBuy.bind(this);
        this.sellNav = this.sellNav.bind(this);
        this.buyNav = this.buyNav.bind(this);
    }

     async componentWillMount(){
       await this.fetchBuyOrders();
       await this.fetchSellOrders();
       await this.fetchBalance();
       await this.fetchPrice();
    }

    async sellNav(){
        this.setState({isMySell:false,mySellNav:"orderBook"});
    }

    async buyNav(){
        this.setState({isMyBuy:false,myBuyNav:"orderBook"});
    }

   async deleteMyOrder(e,i){
        const details = {
            userId:this.props.userId,
            type:i.type,
            orderId:i.orderId
        }
        axios.post('https://api.greyzdorf.io/trade/api/v1/delete',details)
        .then(res=>{
            if(i.type=="Sell"){
                toast(res.data.message, {
                    position: toast.POSITION.TOP_RIGHT,
                });
                this.fetchMySell();
            }
            else{
                toast(res.data.message, {
                    position: toast.POSITION.TOP_RIGHT,
                });
                this.fetchMyBuy();
            }
        })
    }

    async fetchMySell(){
        this.setState({isMySellLoading:true, mySellNav:'myOrders',isMySell:true});
        const details = {
            userId:this.props.userId,
            type:"Sell"
        }   
        axios.post('https://api.greyzdorf.io/trade/api/v1/orders',details)
        .then(res=>{
            let mySellOrders = this.state.mySellOrders;
            mySellOrders=[];
            let count=0;
         if(res.data.length==0){
            this.setState({isMySellLoading:false});
        }
    
       else {
            for(let i=0;i<res.data.length;i++){
                count++;
                if(count<30 && res.data[i].counterAsset == this.props.asset){
                let Temptotal = parseFloat(res.data[i].price*res.data[i].amount).toFixed(8); 
                let TempAmount = parseFloat(res.data[i].amount).toFixed(8);
                let TempPrice = parseFloat(res.data[i].price).toFixed(8);
                const details ={
                    type:"Sell",
                    orderId:res.data[i].orderId
                }
                mySellOrders.push(
                    <tr>
                        <td className="order-red">{TempPrice}</td>
                        <td>{ TempAmount }</td>
                        <td> { Temptotal } <FaRegTrashAlt onClick={(e)=>{this.deleteMyOrder(e,details)}} className="order-red"/></td>
                    </tr>
                    )
            }
            this.setState({mySellOrders,isMySellLoading:false});
        }
        }
        })
        await this.fetchBalance();
    }

    async fetchMyBuy(){
        this.setState({isMyBuyLoading:true, myBuyNav:'myOrders',isMyBuy:true});
        const details = {
            userId:this.props.userId,
            type:"Buy"
        }   
        axios.post('https://api.greyzdorf.io/trade/api/v1/orders',details)
        .then(res=>{
            let myBuyOrders = this.state.myBuyOrders;
            myBuyOrders = [];
            let count=0;
         if(res.data.length==0){
            this.setState({myBuyOrders,isMyBuyLoading:false});
        }
    
       else {
            for(let i=0;i<res.data.length;i++){
                count++;
                if(count<30 && res.data[i].counterAsset == this.props.asset){
                let Temptotal = parseFloat(res.data[i].price*res.data[i].amount).toFixed(8); 
                let TempAmount = parseFloat(res.data[i].amount).toFixed(8);
                let TempPrice = parseFloat(res.data[i].price).toFixed(8);
                const details ={
                    type:"Buy",
                    orderId:res.data[i].orderId
                }
                myBuyOrders.push(
                    <tr>
                        <td className="order-green">{TempPrice}</td>
                        <td>{ TempAmount }</td>
                        <td>{ Temptotal } <FaRegTrashAlt onClick={(e)=>{this.deleteMyOrder(e,details)}} className="order-green"/></td>
                    </tr>
                    )
            }
            this.setState({myBuyOrders,isMyBuyLoading:false});
        }
        }
        })
       await this.fetchBalance();
    }

    fetchPrice(){
        const details = {
            counterAsset:this.state.asset
        }
        axios.post('https://api.greyzdorf.io/trade/api/v1/oneDay',details)
        .then(res=>{
            let i = res.data.length;
            if(i>1){
            let prevPrice = res.data[res.data.length-2].high;
            let lastPrice = res.data[res.data.length-1].high;
            this.setState({high:res.data[i-1].high,low:res.data[i-1].low,volume:res.data[i-1].volume})
            if(lastPrice>prevPrice){
                let TempChange = parseFloat((lastPrice-prevPrice)/prevPrice).toFixed(3);
                if(TempChange!='Infinity'){
                this.setState({isHigh:true,change:TempChange});
                }
                else{
                    this.setState({isHigh:true,change:0}); 
                }
            }
            else{
                let TempChange = parseFloat((prevPrice-lastPrice)/lastPrice).toFixed(3);
                this.setState({isHigh:false,change:TempChange});  
            }
        }
        else{
            this.setState({high:0,low:0,volume:0,change:0});
        }
    })
    }

    getBuyOrders = res => {
        let count=0;
        let buyOrders = this.state.buyOrders;
        buyOrders = [];
            for(let i=0;i<res.length;i++){
                if(res[i].counterAsset == this.state.asset){
                count++;
                if(count<30){
                let Temptotal = parseFloat(res[i].price*res[i].amount).toFixed(8);
                let TempAmount = parseFloat(res[i].amount).toFixed(8);
                let TempPrice = parseFloat(res[i].price).toFixed(8);
                buyOrders.push(
                    <tr>
                        <td className="order-green">{TempPrice}</td>
                        <td>{TempAmount}</td>
                        <td> { Temptotal } </td>
                    </tr>
                    )
                }
            }
        }
        this.setState({buyOrders});  
       }

      getSellOrders = res => {
        let count=0;
        let sellOrders = this.state.sellOrders;
        sellOrders = [];
            for(let i=0;i<res.length;i++){
                if(res[i].counterAsset == this.state.asset){
                count++;
                if(count<30){
                let Temptotal = parseFloat(res[i].price*res[i].amount).toFixed(8);
                let TempAmount = parseFloat(res[i].amount).toFixed(8);
                let TempPrice = parseFloat(res[i].price).toFixed(8);
                sellOrders.push(
                    <tr>
                        <td className="order-red">{TempPrice}</td>
                        <td>{TempAmount}</td>
                        <td> { Temptotal } </td>
                    </tr>
                    )
        }  }
        this.setState({sellOrders});  
      }}

    sendSocketIO() {
        socket.on("sell_orderBook", this.getSellOrders)
        socket.on("buy_orderBook", this.getBuyOrders)
    }
    
    componentDidMount(){
        socket = io(this.state.endpoint,connectionOptions);
        this.sendSocketIO();
        this.fetchBalance();
    }

    async fetchBalance(){
        const details = {
            userId:this.state.userId,
            counterAsset:this.state.asset
        }
        axios.post('https://api.greyzdorf.io/balance/api/v1/eachBalance',details)
        .then(res=>{
            this.setState({
                greyzBal:res.data.GreyzBalance,
                assetBal:res.data.assetBalance
            })
        })
    }

    async fetchBuyOrders(){
        const details = {
            counterAsset:this.props.asset
        }

        axios.post('https://api.greyzdorf.io/trade/api/v1/buyOrders',details)
        .then(res=>{
            let buyOrders = this.state.buyOrders;
            let count=0;
            buyOrders=[];
         if(res.data.length==0){
                this.setState({Buyloading:false});
        }
    
       else {
            for(let i=0;i<res.data.length;i++){
                count++;
                if(count<30){
                let Temptotal = parseFloat(res.data[i].price*res.data[i].amount).toFixed(8); 
                let TempAmount = parseFloat(res.data[i].amount).toFixed(8);
                let TempPrice = parseFloat(res.data[i].price).toFixed(8);
                buyOrders.push(
                    <tr>
                        <td className="order-green">{TempPrice}</td>
                        <td>{TempAmount}</td>
                        <td> { Temptotal } </td>
                    </tr>
                    )
            }
            this.setState({buyOrders,Buyloading:false});
        }
        }
        })
    }

    async fetchSellOrders(){
        const details = {
            counterAsset:this.props.asset
        }

        axios.post('https://api.greyzdorf.io/trade/api/v1/sellOrders',details)
        .then(res=>{
            let sellOrders = this.state.sellOrders;
            let count=0;
            sellOrders=[];
            if(res.data.length==0){
                this.setState({Sellloading:false});
            }    
            else{
            for(let i=0;i<res.data.length;i++){
                count++;
                if(count<30){
                let Temptotal = parseFloat(res.data[i].price*res.data[i].amount).toFixed(8);
                let TempAmount = parseFloat(res.data[i].amount).toFixed(8);
                let TempPrice = parseFloat(res.data[i].price).toFixed(8);
                sellOrders.push(
                    <tr>
                        <td className="order-red">{ TempPrice }</td>
                        <td>{ TempAmount }</td>
                        <td> { Temptotal } </td>
                    </tr>
                    )
                }
                this.setState({sellOrders,Sellloading:false});
                }
        }
        })
    }


    handlePairs(e){
        return(
            window.location=`/exchange/${this.state.asset}/${this.state.userId}`
        )
    }

    handleChange(e){
        e.preventDefault();
        this.setState({[e.target.name]:e.target.value,buyError:false,sellError:false});
    }

    
    handleClick(e,i){
        var location = i;
        window.location = `/orderBook/${this.state.asset}/${this.state.userId}`;
    }
    
    handleOrders(e,i){
        var location = i;
        window.location = `/executedOrders/${this.state.asset}/${this.state.userId}`;
    }

    async submitBuyOrder(e){
        e.preventDefault();
        this.setState({buyButton:true});
        const details = {
            buyerId:this.state.userId,
            counterAsset:this.state.asset,
            amount:parseFloat(this.state.Buyamount).toFixed(8),
            price:parseFloat(this.state.Buyprice).toFixed(8)
        }
        let TempTotal = details.amount*details.price;
        if(details.amount != 0 && this.state.greyzBal > TempTotal ){
        axios.post('https://api.greyzdorf.io/trade/api/v1/buy',details)
        .then(async res=>{
            if(res.data.success=="success"){
                await this.fetchBalance();
                toast(`Order placed at ${details.price} for ${details.amount} ` , {
                    position: toast.POSITION.TOP_RIGHT,
                });
                this.setState({buyButton:false})
            }
            else{
                this.setState({buyError:true,buyErrorMsg:res.data.success,buyButton:false})
            }
        })
    }
    else{
        toast('Error Input', {
            position: toast.POSITION.TOP_RIGHT,
        });
        this.setState({buyError:true,buyErrorMsg:"Error Input",buyButton:false})
    }
    }

    async submitSellOrder(e){
        e.preventDefault();
        this.setState({sellButton:true});
        const details = {
            sellerId:this.state.userId,
            counterAsset:this.state.asset,
            amount:parseFloat(this.state.Sellamount).toFixed(8),
            price:parseFloat(this.state.Sellprice).toFixed(8)
        };
        if(details.amount != 0 && this.state.assetBal > details.amount ){
        axios.post('https://api.greyzdorf.io/trade/api/v1/sell',details)
        .then(async res=>{
            if(res.data.success=="success"){
                toast(`Order placed at ${details.price} for ${details.amount} ` , {
                    position: toast.POSITION.TOP_RIGHT,
                });
                this.setState({sellButton:false})
            }
            
            else{
                this.setState({sellError:true,sellErrorMsg:res.data.success,sellButton:false})
            }
        })
    }  
    else{
        toast(`Error placing order` , {
            position: toast.POSITION.TOP_RIGHT,
        });
        this.setState({sellError:true,sellErrorMsg:"Error Input",sellButton:false})
    }
    await this.fetchBalance();
    }

    componentWillUnmount(){
        socket.emit('disconnect');
    }


    render(){
        return(
            <div className="padding-b">
                <ToastContainer autoClose={2000} style={styles}/>
                <Row>
                    <Col sm={3} className="padding-t-10 text-center">
                    <Nav variant="tabs" defaultActiveKey={this.state.mySellNav} className="orderBookMenu">
                        <Nav.Item>
                            <Nav.Link eventKey="orderBook" onClick={this.sellNav}>Sell Orders</Nav.Link>
                        </Nav.Item>
                        <Nav.Item>
                            <Nav.Link eventKey="myOrders" onClick={this.fetchMySell}>My Orders</Nav.Link>
                        </Nav.Item>
                    </Nav>
                   {this.state.isMySell ? (
                       <Row className="margin-l order-sell">
                       {this.state.isMySellLoading ? ( 
                       <Table size="sm">
                               <thead>
                                   <tr>
                                   <th>Price(GREYZ)</th>
                                   <th>Amount({this.state.asset})</th>
                                   <th>Total(GREYZ)</th>
                                   </tr>
                               </thead>                          
                           <tbody>
                            <td><Skeleton count={10}/></td>      
                            <td><Skeleton count={10}/></td>      
                            <td><Skeleton count={10}/></td>      
                           </tbody>                       
                           </Table>
                            ) : (
                               <Table size="sm">   
                               <thead>
                                   <tr>
                                   <th>Price(GREYZ)</th>
                                   <th>Amount({this.state.asset})</th>
                                   <th>Total(GREYZ)</th>
                                   </tr>
                                   </thead>
                                   <tbody className="sell">
                                    {this.state.mySellOrders}                            
                                   </tbody>                       
                               </Table>
                            )}
                       </Row>
                   ) : (
                <div>
                   <h6 className="text-center padding-t-10">Sell Orders <t /><Link className="more-btn" onClick={(e)=>{this.handleClick(e,"APR")}} >More ></Link></h6>   
                    <Row className="margin-l order-sell">
                        {this.state.Sellloading ? ( 
                        <Table size="sm">
                                <thead>
                                    <tr>
                                    <th>Price(GREYZ)</th>
                                    <th>Amount({this.state.asset})</th>
                                    <th>Total(GREYZ)</th>
                                    </tr>
                                </thead>                          
                            <tbody>
                             <td><Skeleton count={10}/></td>      
                             <td><Skeleton count={10}/></td>      
                             <td><Skeleton count={10}/></td>      
                            </tbody>                       
                            </Table>
                             ) : (
                                <Table size="sm">   
                                <thead>
                                    <tr>
                                    <th>Price(GREYZ)</th>
                                    <th>Amount({this.state.asset})</th>
                                    <th>Total(GREYZ)</th>
                                    </tr>
                                    </thead>
                                    <tbody className="sell">
                                     {this.state.sellOrders}                            
                                    </tbody>                       
                                </Table>
                             )}
                        </Row>
                        </div>
                        )} 
                    </Col>
                    <Col sm={6} className="padding-t-20">
                     <h3 className="text-center">{this.state.asset}/GREYZ</h3>
                     <h6 className="padding-t-0 padding-b-10">Executed Orders <Link className="more-btn" onClick={(e)=>{this.handleOrders(e,"APR")}} >More ></Link></h6>
                        <Row className="text-center stats padding-b-20 justify-content-md-center"> 
                                <Col md={2}>
                                    24H Change<br />
                                    {this.state.isHigh ? (<span className="order-green"> {this.state.change}  </span>) : (<span className="order-red"> {this.state.change}  </span>)}
                                </Col>
                                <Col md={2}>24H High<br />
                                {this.state.high} GREYZ
                                </Col>
                                <Col md={2}>24H Low<br />
                                {this.state.low} GREYZ
                                </Col>
                                <Col md={2}>24H Volume<br />
                                {this.state.volume} GREYZ
                                </Col>
                            </Row>
                    <Container>
                           <Candle asset={this.state.asset} className="padding-t-20"/>
                    </Container>
                        <Row className="padding-b-20 padding-t-20 justify-content-md-right">
                            <Col xs lg="6" className="buy-form ">
                            <Row className="padding-b-20">
                                <Col sm={4}><h6>Buy {this.state.asset}</h6></Col>
                                <Col sm={8}><FaWallet /> - <span className="order-p"> {this.state.greyzBal} GREYZ </span></Col>
                            </Row>
                            {this.state.buyError ? (<Alert variant="primary">{this.state.buyErrorMsg}</Alert>) : (<div></div>)}
                            <Form onSubmit={this.submitBuyOrder}>
                            <Form.Group as={Row} >
                                <Form.Label column sm={3}>
                                <span className="order-p">Price </span>
                                </Form.Label>
                                <Col sm={9}>
                                <Form.Control className="withdraw-form" name="Buyprice" value={this.state.value} onChange={this.handleChange}/>
                                </Col>
                            </Form.Group>

                            <Form.Group as={Row} >
                                <Form.Label column sm={3}>
                                <span className="order-p">Amount </span>
                                </Form.Label>
                                <Col sm={9}>
                                <Form.Control className="withdraw-form" name="Buyamount" value={this.state.value} onChange={this.handleChange} />
                                </Col>
                            </Form.Group>

                            <Form.Group as={Row}>
                                <Form.Label column sm={3} disabled>
                                <span className="order-p">Total </span>
                                </Form.Label>
                                <Col sm={9}>
                                {parseFloat(this.state.Buyamount*this.state.Buyprice).toFixed(8)}
                                </Col>
                            </Form.Group>

                            <Row>
                                <Col md={{ span: 6, offset: 3 }}>
                                {this.state.buyButton ? (<Button type="submit" block disabled> <ScaleLoader height={15} size={3} color={"#ffffff"}/></Button>) : (<Button type="submit" block >Buy Now</Button>)}
                                </Col>
                            </Row>      
                            </Form>

                            </Col>
                            <Col  xs lg="6" className="buy-form">
                            <Row className="padding-b-20">
                                <Col sm={4}><h6>Sell {this.state.asset}</h6></Col>
                                <Col sm={8}><FaWallet /> - <span className="order-p">{this.state.assetBal} {this.state.asset}</span></Col>
                            </Row>
                            {this.state.sellError ? (<Alert variant="primary">{this.state.sellErrorMsg}</Alert>) : (<div></div>)}
                            <Form onSubmit={this.submitSellOrder}>
                            <Form.Group as={Row} >
                                <Form.Label column sm={3}>
                                <span className="order-p">Price </span>
                                </Form.Label>
                                <Col sm={9}>
                                <Form.Control className="withdraw-form" name="Sellprice" value={this.state.value} onChange={this.handleChange}/>
                                </Col>
                            </Form.Group>

                            <Form.Group as={Row} >
                                <Form.Label column sm={3}>
                                <span className="order-p">Amount </span>
                                </Form.Label>
                                <Col sm={9}>
                                <Form.Control className="withdraw-form" name="Sellamount" value={this.state.value} onChange={this.handleChange} />
                                </Col>
                            </Form.Group>

                            <Form.Group as={Row}>
                                <Form.Label column sm={3}>
                                <span className="order-p">Total </span>
                                </Form.Label>
                                <Col sm={9}>
                                {parseFloat(this.state.Sellamount*this.state.Sellprice).toFixed(8)}
                                </Col>
                            </Form.Group>

                            <Row>
                                <Col md={{ span:6, offset: 3 }}>
                                {this.state.sellButton ? (<Button type="submit" block disabled> <ScaleLoader height={15} size={3} color={"#ffffff"}/></Button>) : (<Button type="submit" block >Sell Now</Button>)}
                                </Col>
                            </Row>      
                            </Form>
                            </Col>
                        </Row>
                    </Col>
                    
                    <Col sm={3} className="padding-t-10">
                    <Nav variant="tabs" defaultActiveKey={this.state.myBuyNav} className="orderBookMenu">
                        <Nav.Item>
                            <Nav.Link eventKey="orderBook" onClick={this.buyNav}>Buy Orders</Nav.Link>
                        </Nav.Item>
                        <Nav.Item>
                            <Nav.Link eventKey="myOrders" onClick={this.fetchMyBuy}>My Orders</Nav.Link>
                        </Nav.Item>
                    </Nav>
                    {this.state.isMyBuy ? (
                         <div>
                         <Row className="margin-l order-buy">
                         {this.state.isMyBuyLoading ? ( 
                         <Table size="sm">
                              <thead>
                                     <tr>
                                     <th>Price(GREYZ)</th>
                                     <th>Amount({this.state.asset})</th>
                                     <th>Total(GREYZ)</th>
                                     </tr>
                                 </thead>   
                             <tbody>
                              <td><Skeleton count={10}/></td>      
                              <td><Skeleton count={10}/></td>      
                              <td><Skeleton count={10}/></td>      
                             </tbody>                       
                             </Table>
                              ) : (
                                 <Table size="sm">
                                      <thead>
                                     <tr>
                                     <th>Price(GREYZ)</th>
                                     <th>Amount({this.state.asset})</th>
                                     <th>Total(GREYZ)</th>
                                     </tr>
                                 </thead>   
                                     <tbody className="buy">
                                      {this.state.myBuyOrders}                            
                                     </tbody>                       
                                 </Table>
                              )}
                         </Row>
                         </div>
                    ) : (
                        <div>
                        <h6 className="text-center padding-t-10">Buy Orders <Link className="more-btn" onClick={(e)=>{this.handleClick(e,"APR")}} >More ></Link></h6>
                        <Row className="margin-l order-buy">
                        {this.state.Buyloading ? ( 
                        <Table size="sm">
                             <thead>
                                    <tr>
                                    <th>Price(GREYZ)</th>
                                    <th>Amount({this.state.asset})</th>
                                    <th>Total(GREYZ)</th>
                                    </tr>
                                </thead>   
                            <tbody>
                             <td><Skeleton count={10}/></td>      
                             <td><Skeleton count={10}/></td>      
                             <td><Skeleton count={10}/></td>      
                            </tbody>                       
                            </Table>
                             ) : (
                                <Table size="sm">
                                     <thead>
                                    <tr>
                                    <th>Price(GREYZ)</th>
                                    <th>Amount({this.state.asset})</th>
                                    <th>Total(GREYZ)</th>
                                    </tr>
                                </thead>   
                                    <tbody className="buy">
                                     {this.state.buyOrders}                            
                                    </tbody>                       
                                </Table>
                             )}
                        </Row>
                        </div>
                        )}
                    </Col>
                </Row>
            </div>
        );
    }

}
