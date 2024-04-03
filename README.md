from pyteal import *

class Ballot(TealType):
    candidate_names = [C"pawan", C"nirman", C"Chandu", C"laado-muji"]

class VotingContract(TealProgram):
    candidates = GlobalSlot(0)
    votes = GlobalSlot(1)

    def __init__(self):
        super().__init__()
        self.candidates = Ballot.candidate_names

        with self.Subroutine("get_candidate_index", TealType.uint64) as subroutine:
            candidate_name = self.arg(0)
            index = ScratchVar(TealType.uint64)
            self.if_(candidate_name == self.candidates[0]).then(
                self.sequence(
                    self.int(0).store(index),
                    subroutine.ret(index)
                )
            )
            self.if_(candidate_name == self.candidates[1]).then(
                self.sequence(
                    self.int(1).store(index),
                    subroutine.ret(index)
                )
            )
            self.if_(candidate_name == self.candidates[2]).then(
                self.sequence(
                    self.int(2).store(index),
                    subroutine.ret(index)
                )
            )
            self.if_(candidate_name == self.candidates[3]).then(
                self.sequence(
                    self.int(3).store(index),
                    subroutine.ret(index)
                )
            )
            self.revert()

        with self.Section("init") as s:
            self.candidates = self.candidates.write(self.candidates.read())
            self.votes = self.votes.write(self.candidates.length().int())

        with self.Section("vote") as s:
            candidate_index = self.arg(0)
            vote_count = self.votes.read(candidate_index).uint64()
            self.votes.write(vote_count.add(self.int(1)), candidate_index)

        with self.Section("check_admin") as s:
            admin = self.current_application_address()
            caller = self.get_caller()
            self.if_(caller != admin).then(
                self.revert()
            )

        with self.Global("vote") as s:
            self.if_(self.block_height() > self.global_get(Int(0))).then(
                self.seq(
                    self.check_admin(),
                    self.vote(self.arg(0))
                )
            )
