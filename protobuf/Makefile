all:
	protoc --python_out=. activity.proto
	protoc --python_out=. segment-result.proto
	protoc --python_out=. profile.proto
	protoc --python_out=. per-session-info.proto
	protoc --python_out=. login-response.proto
	protoc --python_out=. periodic-info.proto
	protoc --python_out=. world.proto
	protoc --python_out=. goal.proto

clean:
	rm -f *_pb2.py *_pb2.pyc
