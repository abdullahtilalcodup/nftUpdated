//SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "hardhat/console.sol";
import "erc721a/contracts/ERC721A.sol";

/// @title NFTA
/// @author Adam Bawany
/// @custom:experimental This is an experimental contract.
contract NFTA is ERC721A, Ownable {
    using Strings for uint256;
    string public baseURI;
    string public baseExtension = ".json";
    uint256 public cost = 0.000003 ether;
    uint256 public maxSupply = 3333;
    uint256 public maxMintAmount = 3;
    bool public paused = false;
    bool public revealed = false;
    string public notRevealedUri;
    mapping(address => bool) whitelistedAddressesO;
    mapping(address => bool) whitelistedAddressesL;
    bool public onlyWhiteListed = false;
    bool public publicSale = false;
    uint256 public nftPerAddressLimitFreeO = 3; //whitelist
    uint256 public nftPerAddressLimitFreeL = 2;
    uint256 public nftPerAddressLimitO = 3;
    uint256 public nftPerAddressLimitL = 3;
    uint256 public nftPerAddressLimitFreePublic = 1;
    uint256 public nftPerAddressLimitPublic = 3;
    uint256 public numberOfAddressesWhitelistedO = 0;
    uint256 public maxNumberOfWhitelistedAddressesO = 20;
    uint256 public numberOfAddressesWhitelistedL = 0;
    uint256 public maxNumberOfWhitelistedAddressesL = 520;
    uint256 public numberOfAddressesPublic = 0;
    uint256 public maxNumberOfAddressesPublic = 1233;

    //   mapping(address => bool) public whitelisted;

    constructor(
        string memory _name,
        string memory _symbol,
        string memory _initBaseURI,
        string memory _initNotRevealedUri
    ) ERC721A(_name, _symbol) {
        setBaseURI(_initBaseURI);
        setNotRevealedURI(_initNotRevealedUri);
        // mint(msg.sender, 20);
    }

    /// @notice Gives the base uri for tocken
    /// @dev use for the BaseUri setup at the time of initialization
    /// @return base uri of tockens on ipfs in string
    function _baseURI() internal view virtual override returns (string memory) {
        return baseURI;
    }

    /// @notice mints the nft
    /// @dev main minting function.can improve via lazy minting
    /// @param _mintAmount is the quantity of tocken to mint
    function mint(uint256 _mintAmount) public payable {
        require(!paused);
        uint256 supply = totalSupply();
        require(_mintAmount > 0);
        require(_mintAmount <= maxMintAmount);
        require(supply + _mintAmount <= maxSupply);
        require(balanceOf(msg.sender) + _mintAmount <= maxMintAmount);

        if (onlyWhiteListed == true) {
            if (
                isWhiteListedO(msg.sender) 
            ) {
                require(
                    isWhiteListedO(msg.sender),
                    "user is not whitelisted in O"
                );
                uint256 ownerTokenCount = balanceOf(msg.sender);
                require(ownerTokenCount < nftPerAddressLimitO);
                uint256 freeMintAvailable = nftPerAddressLimitFreeO -
                    ownerTokenCount;
                if (_mintAmount <= freeMintAvailable) {
                    _safeMint(msg.sender, _mintAmount);
                } else {
                    _safeMint(msg.sender, freeMintAvailable);
                    uint256 ownerTokenCountNewBalance = balanceOf(msg.sender);
                    uint256 paidMint = nftPerAddressLimitO -
                        ownerTokenCountNewBalance;
                    require(msg.value >= paidMint * cost);

                    _safeMint(msg.sender, paidMint);
                }
            } else if (
                isWhiteListedL(msg.sender) 
            ) {
                require(
                    isWhiteListedL(msg.sender),
                    "user is not whitelisted in L"
                );
                uint256 ownerTokenCount = balanceOf(msg.sender);
                require(ownerTokenCount < nftPerAddressLimitL, "limit exceed");
                uint256 freeMintAvailable = nftPerAddressLimitFreeL -
                    ownerTokenCount;
                if (_mintAmount <= freeMintAvailable) {
                    _safeMint(msg.sender, _mintAmount);
                } else {
                    uint256 paidMintCheck = _mintAmount - freeMintAvailable;
                    require(msg.value >= paidMintCheck * cost, "payment");

                    _safeMint(msg.sender, _mintAmount);
                }
            } else {
                require(msg.value >= _mintAmount * cost, "payment");
                for (uint256 i = 1; i <= _mintAmount; i++) {
                    _safeMint(msg.sender, supply + i);
                }
            }
        } else {
            uint256 ownerTokenCount = balanceOf(msg.sender);
            require(ownerTokenCount < nftPerAddressLimitPublic, "limit exceed");

            uint256 freeMintAvailable = nftPerAddressLimitFreePublic >
                ownerTokenCount
                ? nftPerAddressLimitFreePublic - ownerTokenCount
                : 0;

            if (_mintAmount <= freeMintAvailable) {
                _safeMint(msg.sender, _mintAmount);
                numberOfAddressesPublic++;
            } else {
                console.log("before paid mint", supply);
                uint256 paidMintCheck = _mintAmount - freeMintAvailable;
                console.log("paid mint", paidMintCheck);
                require(msg.value >= paidMintCheck * cost, "payment");

                _safeMint(msg.sender, _mintAmount);
                console.log("after paid mint");
            }
        }
    }

    /// @notice check if the address is in whitelisted og list
    /// @dev use to check if address is in whitelisted og list
    /// @param _user is the address of wallet to check
    /// @return it returns the boolen,true if address exist in the whitelist og list
    function isWhiteListedO(address _user) public view returns (bool) {
        bool userIsWhitelisted = whitelistedAddressesO[_user];
        return userIsWhitelisted;
    }

    /// @notice check if the address is in whitelisted l list
    /// @dev use to check if address is in whitelisted L list
    /// @param _user is the address of wallet to check
    /// @return it returns the boolen,true if address exist in the whitelist l list
    function isWhiteListedL(address _user) public view returns (bool) {
        bool userIsWhitelisted = whitelistedAddressesL[_user];
        return userIsWhitelisted;
    }

    /// @notice gives the token uri of specific token
    /// @dev it is to give us the token uri when minted an nft
    /// @param tokenId is the index of the token for which uri has to return
    /// @return it returns the uri of specific token
    function tokenURI(uint256 tokenId)
        public
        view
        virtual
        override
        returns (string memory)
    {
        require(
            _exists(tokenId),
            "ERC721Metadata: URI query for nonexistent token"
        );

        if (revealed == false) {
            return notRevealedUri;
        }

        string memory currentBaseURI = _baseURI();
        return
            bytes(currentBaseURI).length > 0
                ? string(
                    abi.encodePacked(
                        currentBaseURI,
                        tokenId.toString(),
                        baseExtension
                    )
                )
                : "";
    }

    /// @notice reveal the nft art
    /// @dev set the reveal to true inorder to reveal nft art.
    function reveal() public onlyOwner {
        revealed = true;
    }

    /// @notice set the cost for paid mint
    /// @dev it is to set the cost for the tokens minted in paid mint
    /// @param _newCost is the new cost to set for the paid mint
    function setCost(uint256 _newCost) public onlyOwner {
        cost = _newCost;
    }

    /// @notice set the max amount of token that can be minted by a simgle wallet
    /// @dev it is to set max amount of token that can be minted by a simgle wallet
    /// @param _newmaxMintAmount is the new max amount that a single wallet can mint totally
    function setmaxMintAmount(uint256 _newmaxMintAmount) public onlyOwner {
        maxMintAmount = _newmaxMintAmount;
    }

    /// @notice it sets the uri for the image to show when tokens are not revealed
    /// @dev it is to set uri of json that has details to show along with image when the tokens are not revealed
    /// @param _notRevealedURI is the new uri of json for the metadata when tokens are not revealed
    function setNotRevealedURI(string memory _notRevealedURI) public onlyOwner {
        notRevealedUri = _notRevealedURI;
    }

    /// @notice it sets the uri for the data including images,description to show when tokens are  revealed
    /// @dev it is to set domain uri of json metadata that has details to show along with image when the tokens are revealed
    /// @param _newBaseURI is the new domain uri of json for the metadata when tokens are  revealed
    function setBaseURI(string memory _newBaseURI) public onlyOwner {
        baseURI = _newBaseURI;
    }

    /// @notice it sets the extension of file containing metadata
    /// @dev it is to set file extension of metadata that has details to show along with image when the tokens are not revealed
    /// @param _newBaseExtension is the new extension of the metadata file
    function setBaseExtension(string memory _newBaseExtension)
        public
        onlyOwner
    {
        baseExtension = _newBaseExtension;
    }

    /// @notice it stops/resume wallet from minting.
    /// @dev it is to stop/resume the minting process.Could be used when system is compromised.
    /// @param _state is the boolean to whther resume or pause the minting process.
    function pause(bool _state) public onlyOwner {
        paused = _state;
    }

    /// @notice it stops/resume the whitelisting minting process
    /// @dev it is to stop/resume the whitelisting minting process.When passed false public mint will be open
    /// @param _state is the boolean to whther resume or pause the whitelisting minting process or public mint.
    function setOnlyWhiteListed(bool _state) public onlyOwner {
        onlyWhiteListed = _state;
    }

    /// @notice it removes the address from whitelist Og list
    /// @dev it removes the address from whitelist Og list.Could be removed in future due  to high gass fee
    /// @param _user is the address of wallet to be removed from whitelist Og list
    function removeWhitelistUsersO(address _user) public onlyOwner {
        // Validate the caller is already part of the whitelist.
        require(
            whitelistedAddressesO[_user],
            "Error: Sender is not whitelisted"
        );
        // Set whitelist boolean to false.
        whitelistedAddressesO[_user] = false;
        // This will decrease the number of whitelisted addresses.
        numberOfAddressesWhitelistedO -= 1;
    }

    /// @notice it removes the address from whitelist L list
    /// @dev it removes the address from whitelist l list.Could be removed in future due  to high gass fee
    /// @param _user is the address of wallet to be removed from whitelist L list
    function removeWhitelistUsersL(address _user) public onlyOwner {
        // Validate the caller is already part of the whitelist.
        require(
            whitelistedAddressesL[_user],
            "Error: Sender is not whitelisted"
        );
        // Set whitelist boolean to false.
        whitelistedAddressesL[_user] = false;
        // This will decrease the number of whitelisted addresses.
        numberOfAddressesWhitelistedL -= 1;
    }

    /// @notice it is to withraw funds from the paid mint
    /// @dev it withdraws the fund from minting to the wallet of owner
    function withdraw() public payable onlyOwner {
        (bool os, ) = payable(owner()).call{value: address(this).balance}("");
        require(os);
    }
}

// ["0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2","0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db"]
