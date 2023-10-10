Using import declarations of the form import {<identifier_name>} from 'some/file.sol' avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation.

Instances (4):

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import "@openzeppelin/contracts/utils/math/Math.sol";
