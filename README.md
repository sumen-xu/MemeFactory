# MemeFactory
MemeFactory
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract MemeToken {
    string public name;
    string public symbol;
    uint8 public decimals = 18;
    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    address public owner;

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    constructor(string memory _name, string memory _symbol, uint256 _initialSupply) {
        name = _name;
        symbol = _symbol;
        totalSupply = _initialSupply * (10 ** decimals);
        balanceOf[msg.sender] = totalSupply;
        owner = msg.sender;
        emit Transfer(address(0), msg.sender, totalSupply);
    }

    function transfer(address to, uint256 amount) public returns (bool) {
        _transfer(msg.sender, to, amount);
        return true;
    }

    function _transfer(address from, address to, uint256 amount) internal {
        require(from != address(0), "Transfer from the zero address");
        require(to != address(0), "Transfer to the zero address");
        require(balanceOf[from] >= amount, "Insufficient balance");
        balanceOf[from] -= amount;
        balanceOf[to] += amount;
        emit Transfer(from, to, amount);
    }

    function approve(address spender, uint256 amount) public returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address from, address to, uint256 amount) public returns (bool) {
        require(allowance[from][msg.sender] >= amount, "Insufficient allowance");
        allowance[from][msg.sender] -= amount;
        _transfer(from, to, amount);
        return true;
    }

    function mint(address to, uint256 amount) public {
        require(msg.sender == owner, "Only owner can mint");
        totalSupply += amount;
        balanceOf[to] += amount;
        emit Transfer(address(0), to, amount);
    }

    function burn(uint256 amount) public {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }

    function transferOwnership(address newOwner) public {
        require(msg.sender == owner, "Only owner");
        require(newOwner != address(0), "New owner is the zero address");
        owner = newOwner;
    }
}

contract BaseMemeFactory {
    address public factoryOwner;
    uint256 public creationFee = 0.001 ether;
    mapping(address => address[]) public tokensCreatedBy;

    event TokenCreated(
        address indexed tokenAddress,
        address indexed creator,
        string name,
        string symbol,
        uint256 totalSupply
    );

    constructor() {
        factoryOwner = msg.sender;
    }

    function createMemeToken(
        string memory name,
        string memory symbol,
        uint256 totalSupply
    ) external payable returns (address) {
        require(msg.value >= creationFee, "Insufficient creation fee");
        MemeToken newToken = new MemeToken(name, symbol, totalSupply);
        tokensCreatedBy[msg.sender].push(address(newToken));
        payable(factoryOwner).transfer(msg.value);
        emit TokenCreated(address(newToken), msg.sender, name, symbol, totalSupply);
        return address(newToken);
    }

    function getTokensCreatedBy(address creator) external view returns (address[] memory) {
        return tokensCreatedBy[creator];
    }

    function updateCreationFee(uint256 newFee) external {
        require(msg.sender == factoryOwner, "Only factory owner");
        creationFee = newFee;
    }
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.34;

contract MemeToken {
    string public name;
    string public symbol;
    uint8 public decimals = 18;
    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    address public owner;

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    constructor(string memory _name, string memory _symbol, uint256 _initialSupply) {
        name = _name;
        symbol = _symbol;
        totalSupply = _initialSupply * (10 ** decimals);
        balanceOf[msg.sender] = totalSupply;
        owner = msg.sender;
        emit Transfer(address(0), msg.sender, totalSupply);
    }

    function transfer(address to, uint256 amount) public returns (bool) {
        _transfer(msg.sender, to, amount);
        return true;
    }

    function _transfer(address from, address to, uint256 amount) internal {
        require(from != address(0), "Transfer from the zero address");
        require(to != address(0), "Transfer to the zero address");
        require(balanceOf[from] >= amount, "Insufficient balance");
        balanceOf[from] -= amount;
        balanceOf[to] += amount;
        emit Transfer(from, to, amount);
    }

    function approve(address spender, uint256 amount) public returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address from, address to, uint256 amount) public returns (bool) {
        require(allowance[from][msg.sender] >= amount, "Insufficient allowance");
        allowance[from][msg.sender] -= amount;
        _transfer(from, to, amount);
        return true;
    }

    function mint(address to, uint256 amount) public {
        require(msg.sender == owner, "Only owner can mint");
        totalSupply += amount;
        balanceOf[to] += amount;
        emit Transfer(address(0), to, amount);
    }

    function burn(uint256 amount) public {
        require(balanceOf[msg.sender] >= amount, "Insufficient balance");
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }

    function transferOwnership(address newOwner) public {
        require(msg.sender == owner, "Only owner");
        require(newOwner != address(0), "New owner is the zero address");
        owner = newOwner;
    }
}

contract BaseMemeFactory {
    address public factoryOwner;
    uint256 public creationFee = 0.001 ether;
    mapping(address => address[]) public tokensCreatedBy;

    event TokenCreated(
        address indexed tokenAddress,
        address indexed creator,
        string name,
        string symbol,
        uint256 totalSupply
    );

    constructor() {
        factoryOwner = msg.sender;
    }

    function createMemeToken(
        string memory name,
        string memory symbol,
        uint256 totalSupply
    ) external payable returns (address) {
        require(msg.value >= creationFee, "Insufficient creation fee");

        MemeToken newToken = new MemeToken(name, symbol, totalSupply);
        tokensCreatedBy[msg.sender].push(address(newToken));

        // 推荐写法（替代 transfer）
        (bool sent, ) = payable(factoryOwner).call{value: msg.value}("");
        require(sent, "Failed to send fee");

        emit TokenCreated(address(newToken), msg.sender, name, symbol, totalSupply);
        return address(newToken);
    }

    function getTokensCreatedBy(address creator) external view returns (address[] memory) {
        return tokensCreatedBy[creator];
    }

    function updateCreationFee(uint256 newFee) external {
        require(msg.sender == factoryOwner, "Only factory owner");
        creationFee = newFee;
    }

    function withdraw() external {
        require(msg.sender == factoryOwner, "Only factory owner");

        // 推荐写法（替代 transfer）
        (bool sent, ) = payable(factoryOwner).call{value: address(this).balance}("");
        require(sent, "Failed to withdraw");
    }
}
    function withdraw() external {
        require(msg.sender == factoryOwner, "Only factory owner");
        payable(factoryOwner).transfer(address(this).balance);
    }
}
createMemeToken(string name, string symbol, uint256 totalSupply)
