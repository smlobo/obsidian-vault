
`transpose`
`(self, permutation)`
-> `tensor`

`broadcast_in_dim`
`(self, [int], ...)`
-> `tensor`
- duplicates/extends data along new or existing axes
- inputs:
  - mapping dimension
  - input shape
  - output shape

`rehsape`
`(self, ...)`
-> `tensor`
- reinterpret buffer layout to new dimensions without altering data ordering or replicating values
- inputs:
  - input shape
  - output shape

`dot_general`
`(lhs, rhs, ...)`
-> `tensor`
- generalized tensor contraction
- inputs:
  - dimensions batched
  - dimensions summed over

