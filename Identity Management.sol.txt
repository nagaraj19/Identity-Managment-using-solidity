/*
Identity Management - Need for identity management arises when someone has to be identified , authenticated 
or authorized for a certain action or access services or systems.
In order to avoid submitting your identity proof in multiple forms and in multiple places, it is advised to 
have a distributed storage of same which can be accessed and/or verified by multiple agencies.

It is proposed, that a smart contract be deployed by Ministry of Human Resources (MHRE) which would be acting
as an administrator (admin) and will have exclusive rights to add agencies which can only access database or 
access, update & verify database records.

An agency will have to offline submit request with :
1. Name of agency
2. Short description.

MHRE will generate a random 4 digit number to identify the agency.

A client will offline submit following credentials :
1. AADHAR Card number - 16 digit number
2. Hash of picture saved on a distributed file system ( like ipfs).

The details can be submitted to an agency which has rights to update database.

Everytime a client is verified by an agency, a 'Like' is added to the client's database, indicating how many
agencies has verified the credentials. This will help in instilling trust in data.

In case, an agency finds the client's data to be malicious, the database will block the client.

Client will have to approach MHRE to re-permit him/her to be allowed to submit correct credentials.

*/

// SPDX-License-Identifier: MIT
pragma solidity 0.8.16;

contract IdentityManagement {

    address MHRE;

    constructor() {
        MHRE = msg.sender;
    }

    struct Client {
        uint aadharNo;
        bytes32 picHash;
        uint like;
    }

    struct Agency {
        uint agencyNo;  //4 digit random code
        string  name ;
        string description;
        bool canUpdate;
    }

    /*
    Mapping a client's ethAddress to the Client struct
    We also keep an array of all keys of the mapping to be able to loop through them when required.
     */
    mapping(address => Client) clientdb;
    address[] clientAddr;
    
    mapping  (address => Agency)public Agencydb ;
    address[] AgencyAddrlist;
    mapping(address=>bool)AgencyAddr;
    mapping(uint =>bool)agencyNolist;
    
    // function addAgency()
    function addAgency(address ag_add,string memory name,string memory description,bool update) public isOwner{
        uint time=block.timestamp;
        uint num=(time%10000);
        uint agencyNo = checkRandom(num);
        Agencydb[ag_add]=Agency(agencyNo,name,description,update);
        AgencyAddr[ag_add]=true;
        AgencyAddrlist.push(ag_add);
        }   

    //function to check random number    
    function checkRandom(uint num)internal  returns(uint){
        while (agencyNolist[num]==true) {
             // while loop
                num=num+1;
                if(num >=10000){
                    num=0;
                }

            }
            agencyNolist[num]=true;
            
            return num;
        }
    
    //check owner such than only MHRE is allowed is allowed to add owner
    modifier isOwner() {

        require(msg.sender == MHRE, "Caller is not owner");
        _;
    }
     //function to view agency
        function viewAgencyPresntinArray (uint index)public  view returns(Agency memory){ 
            address agency_add=AgencyAddrlist[index];
            return Agencydb[agency_add];


     }
     //function to add client based by agency
    function addclient(address client_add,uint adhar,bytes32 hashpic)public  {
        require(AgencyAddr[msg.sender]); // only registered agency can add the client 
        require(Agencydb[msg.sender].canUpdate==true);//regiesterd agency should have update permission

        clientdb[client_add]=Client(adhar,hashpic,1);
        clientAddr.push(client_add);

    }
    //check the client details based on address for registered agency 
    function viewclient(address client) public view returns(Client memory){
        require(AgencyAddr[msg.sender]);
        return clientdb[client];

    }
    //check the client data and match
    function validateClientData(address client,uint adhar,bytes32 hashpic) public returns(uint){
       uint ad=clientdb[client].aadharNo;
       bytes32 hash1=clientdb[client].picHash;
       if(ad==adhar && hash1==hashpic){      
         clientdb[client].like= clientdb[client].like+1;
         return(clientdb[client].like);

       }
       else{
           clientdb[client].like= clientdb[client].like-1;
            return(clientdb[client].like);

       }

    }
    //add client data by MHRE
    function permitReintroduction(address client_add,uint adhar,bytes32 hashpic)public isOwner {
        require(clientdb[client_add].like<=0);
        clientdb[client_add]=Client(adhar,hashpic,1);

    }


}
    





