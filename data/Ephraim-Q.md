# Lack of Zero Address Check in _transfer Function

With the ability of users being able to send/gift/distribute Curve token to another address using the transferCurvesToken and transferAllCurvesTokens, this allows for users to pass in the to parameters for who would receive these tokens. However, this functions call the principal _transfer function that updates all involved state variables but this function misses a critical check that validates that the to address is not an address zero.

Users who gift unknowingly or mistakenly to address zero cannot retrieve such token anymore.