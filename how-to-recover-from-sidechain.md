v0.1.14 enforced the total difficulty check. if the block found by node , the total diff is less than the main net, the new block propagation will be rejected.

Steps to identify if node are in sidechain.

    turn on p2p "show-log" in config.xml
	<p2p>
			<ip>127.0.0.1</ip>
			<port>30303</port>
			<discover>true</discover>
			<show-log>true</show-log>
			<show-status>true</show-status>
	</p2p>


    find "p2p-status" section inside log file.

[[/images/p2p-status.png| p2p status]]

    find your node id in p2p-status.

    compare your block number and total diff with all other nodes.
    if your block number >= others but total diff < other's total diff, you are in side chain.

Steps to recover.

    revert your db to most reasonable block number.
    ./aion.sh -r "your_block_number_when_forked"

    restart kernel to check "p2p-status" again to confirm your db back to main chain.


